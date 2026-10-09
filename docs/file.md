# file
##
### codemeta.schema
・CodeMeta 3.0 (https://w3id.org/codemeta/3.0) に準拠した codemeta.json 用の JSON Schema
・作成時session:https://claude.ai/chat/9327d1e2-4488-48c8-9853-73a0fa8fd1d3

補足:
mode / prot / user / uid / group / gid / inode / dev / time といった環境依存・実行時依存の項目は、説明の管理に不要でノイズ（差分・環境差）になるため意図的に定義していない。（未定義のプロパティは unevaluatedProperties:false によりエラーになる）
ユースケース:
・使用するパッケージマネージャ自体（pnpm や yarn など）を明示したい場合は、softwareSuggestions に入れるべき
## desc
### DIRDESC.yaml