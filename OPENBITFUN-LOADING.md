# OpenBitFun 加载验证说明

## 实际加载入口

OpenBitFun 个人助理会话绑定一个 `workspacePath`，并从该工作区**根目录**读取：

```text
IDENTITY.md
SOUL.md
USER.md
```

因此，`digital-human-profile/` 是版本化源目录；运行中的助理使用的是实际 workspace 根目录中的同名文件，而不是自动读取嵌套目录。

当前 SivenBot 实际工作区：

```text
/Users/zhengxiwan/.openbitfun/personal_assistant/workspace
```

已同步到运行时的文件：

```text
workspace/IDENTITY.md
workspace/SOUL.md
workspace/USER.md
workspace/skills/sivenbot/SKILL.md
```

## 最小只读验证

### 1. 检查运行时文件存在

```bash
ROOT="/Users/zhengxiwan/.openbitfun/personal_assistant/workspace"
for f in IDENTITY.md SOUL.md USER.md skills/sivenbot/SKILL.md; do
  test -f "$ROOT/$f" && echo "present: $f" || exit 1
done
```

### 2. 确认会话绑定的工作区

```bash
python3 - <<'PY'
import json

p = "/Users/zhengxiwan/.openbitfun/projects/b139f269d129cec2d0bf3551/sessions/8cdc0c4b-65ac-4578-8b4c-efa4ba78cecb/metadata.json"
d = json.load(open(p))
print("workspacePath:", d.get("workspacePath"))
print("projectWorkspacePath:", d.get("projectWorkspacePath"))
print("executionTarget.rootPath:", d.get("executionTarget", {}).get("rootPath"))
PY
```

预期三个路径都指向实际助理工作区：

```text
/Users/zhengxiwan/.openbitfun/personal_assistant/workspace
```

### 3. 启动只读加载验证会话

在该工作区创建一个 Standard 会话，并要求它只读取：

```text
SOUL.md、IDENTITY.md、USER.md、skills/sivenbot/SKILL.md
```

验证它能回答以下四点：

1. 助理名称和定位是 `SivenBot`、siven 的 AI 数字分身与协作助手；
2. 默认报告包含：结论、当前状态、已完成、已验证、未验证、风险或阻塞、下一步；
3. 工程链路是：管理入口 → 项目管理 → 项目自有执行 → 项目报告 → 全局汇总 → 调度回执；
4. 仅凭历史测试、没有当前生产回执和明确发布授权时，不直接发版或通知客户。

验证会话应明确声明：未修改文件、未执行外部动作。

## 已完成验证

- 版本源和运行时四个文件的 SHA-1 一致；
- 运行时文件存在性检查通过；
- 当前会话元数据的 `workspacePath` 和 `executionTarget.rootPath` 指向上述工作区；
- 只读 Standard 验证会话成功读取四个文件，并正确返回身份、报告字段、工程链路和发布授权边界；
- 验证会话未修改文件，也未执行外部动作。

## 注意事项

- `SOUL.md`、`IDENTITY.md`、`USER.md` 是 OpenBitFun 助理的人设入口；
- `skills/sivenbot/SKILL.md` 是按场景使用的 Skill，不应把它与三份核心人设文件混为一谈；
- 人设文件不等于工具权限。`.openbitfun/config/tool_permissions.json` 仍负责工具访问边界；
- 如果切换到其他 `workspace-<id>` 助理，必须将三份核心文件和需要的 Skill 同步到那个助理工作区后再验证；
- 修改版本源后，需要重新同步到实际 workspace，并重新执行最小验证。
