该 `is_valid` 是一个局部校验函数：根据 `validator` 的类型检查参数 `val` 是否符合要求；成功时不返回值，失败时返回错误信息字符串。第 1121 行是函数入口，关键点是它同时接收参数名、待校验值、校验器、自定义提示，以及是否允许类型别名。

```lua
-- /home/allworldg/neovim/runtime/lua/vim/_core/shared.lua:1121
1121:   local function is_valid(param_name, val, validator, message, allow_alias)
1122:     if type(validator) == 'string' then
1123:       local expected = allow_alias and type_aliases[validator] or validator
1124:       -- 当 validator 是字符串时，它表示期望类型；allow_alias 为真时会先解析类型别名。
1125:
1126:       if not expected then
1127:         return string.format('invalid type name: %s', validator)
1128:         -- 类型名或别名无法解析，返回“无效类型名”错误。
1129:       end
1130:
1131:       if not is_type(val, expected) then
1132:         return ('%s: expected %s, got %s'):format(param_name, message or expected, type(val))
1133:         -- 待校验值类型不匹配时，返回包含参数名、期望类型、实际类型的错误。
1134:       end
1135:     elseif vim.is_callable(validator) then
1136:       -- Check user-provided validation function
1137:       local valid, opt_msg = validator(val)
1138:       -- 当 validator 可调用时，用它作为自定义校验函数；返回值 valid 决定是否通过。
1139:       if not valid then
1140:         local err_msg = ('%s: expected %s, got %s'):format(
1141:           param_name,
1142:           message or '?',
1143:           tostring(val)
1144:         )
1145:         err_msg = opt_msg and ('%s. Info: %s'):format(err_msg, opt_msg) or err_msg
1146:         -- 自定义校验失败时，可把校验函数返回的 opt_msg 追加到错误信息中。
1147:
1148:         return err_msg
1149:       end
1150:     elseif type(validator) == 'table' then
1151:       for _, t in ipairs(validator) do
1152:         local expected = allow_alias and type_aliases[t] or t
1153:         -- 当 validator 是表时，表示允许多个类型，逐个尝试匹配。
1154:         if not expected then
1155:           return string.format('invalid type name: %s', t)
1156:         end
1157:
1158:         if is_type(val, expected) then
1159:           return -- success
1160:           -- 任一类型匹配即成功；成功路径返回 nil。
1161:         end
1162:       end
1163:
1164:       -- Normalize validator types for error message
1165:       if allow_alias then
1166:         for i, t in ipairs(validator) do
1167:           validator[i] = type_aliases[t] or t
1168:         end
1169:       end
1170:       -- 注意：这里会就地修改 validator 表，把别名替换为实际类型，用于后续错误信息。
1171:
1172:       return string.format(
1173:         '%s: expected %s, got %s',
1174:         param_name,
1175:         table.concat(validator, '|'),
1176:         type(val)
1177:       )
1178:       -- 所有允许类型都不匹配时，返回形如 “name: expected a|b, got c” 的错误。
1179:     else
1180:       return string.format('invalid validator: %s', tostring(validator))
1181:       -- validator 既不是字符串、可调用对象、也不是表，则校验器本身无效。
1182:     end
1183:   end
```

主要逻辑分三类：字符串校验单一类型；可调用对象执行自定义校验；表校验多个允许类型。第 1121 行相关的核心信息是 `allow_alias` 会影响字符串和表形式的类型解析，`message` 只在部分错误信息中覆盖默认期望描述。成功时该函数返回 `nil`，失败时返回错误字符串。
