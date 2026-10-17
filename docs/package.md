#
## 基本情報
### context
構造化されたパッケージの説明を記すには*package.json*だけでは十分に表現することができず、
一般的な解決手法は*`README.md`か`docs`を情報源にすることである。

この課題に対してpackage情報の整合性確保・構造化を目的に、私はこれを作った。
### spec
desc:package情報の整合性確保・構造化を目的としたschema設計。これを情報源にREADME.mdを生成することも可能

README.md を「完全かつ構造化された情報源」にを根拠にする為のデータモデルの確保。
`diff`や`json-patch`のような役割とは被らない。
`パッケージの情報源`をREADME.mdなどの自然言語にすることは許されない。(構造化された情報を元に作られた README.md ならよい)

## codemeta.json

## `package.cdx.json`
### 一部プロパティの使い分け(cdx準拠)
pedigree.patches[number].diff//変更のパッチファイル
pedigree.patches[number].resolves[number]//変更の意味・役割の説明
### 追加制約(cdxからの)
pedigree.patches[number]
において

・pedigree.patches[number].resolves[0]
or
・pedigree.patches[number].diff
が必須
### 難点と対応
package.cdx.json::Component.pedigree.patches[number]
がどの
package.cdx.json::Component.pedigree.ancestors[number]
に対しての操作かは、標準のプロパティでは機械的に判断がつかない。
(CycloneDX の patch は ancestor を参照するプロパティを持たず、vendor schema は `additionalProperties:false` のため拡張プロパティも置けない。)

決定(現時点):
・拡張プロパティ(`ref` など)は**追加しない**。標準の範囲だけで運用する。
・必要なのは「意味的にどんな差分か(`resolves`)」であり、「どの ancestor への差分か」ではない。
・加えて可能なら「機械可読な差分(`diff`:JSON Patch など)」を記す。
・ancestor が1つなら対応は自明。複数 ancestor で対応が必要になった時点で、改めて方針を決める。
### examples

```json example
{
	"$schema": "https://raw.githubusercontent.com/rin00159/ripo-schema/refs/heads/main/schema/0.0.0/package.cdx.schema.json",
	"bomFormat": "CycloneDX",
	"specVersion": "1.6",
	"metadata": {
		"component": {
			"type": "library",
			"name": "openmath-jsonschema",
			"licenses": [{ "license": { "id": "MIT" } }]
		}
	},
	"components": [
		{
			"type": "file",
			"name": "src/openmath.json",
			"licenses": [{ "license": { "id": "CC-BY-3.0" } }],
			"pedigree": {
				"ancestors": [
					{
						"type": "file",
						"name": "src/schema/openmath.json",
						"description": "OpenMath-JSON: JSON Schema for OpenMath objects",
						"copyright": "Copyright (c) Tom Wiesing",
						"authors": [{ "name": "Tom Wiesing" }],
						"licenses": [
							{
								"license": {
									"id": "CC-BY-3.0",
									"url": "https://creativecommons.org/licenses/by/3.0/"
								}
							}
						],
						"externalReferences": [
							{
								"type": "vcs",
								"url": "https://github.com/tkw1536/OpenMath-JSON"
							}
						]
					}
				],
				"patches": []
			}
		}
	]
}
```
```json:patch jsonpatchファイル参照
"patches": [
	{
		"type": "unofficial",
		"diff": { "url": "patches/openmath.json-patch.json" }
	}
]
//# patches/openmath.json-patch.json
[{"op":"remove","path":"/…/exclusiveMaximum"},{"op":"remove","path":"/…/exclusiveMinimum"}]
```
```json:patch ancestor を bom-ref で識別しつつ、jsonpatchファイル参照
"pedigree": {
	"ancestors": [
		{
			"bom-ref": "openmath.json",
			"type": "file",
			"name": "src/schema/openmath.json"
		}
	],
	"patches": [
		{
			"type": "unofficial",
			"diff": { "url": "patches/openmath.json-patch.json" }
		}
	]
}
```
//patch → ancestor の対応付けは行わない(上記「決定」参照)。

```json:patch jsonpatchファイル保持
"patches": [
	{
		"type": "unofficial",
		"diff": {
			"text": {
				"contentType": "application/json-patch+json",
				"content": "[{\"op\":\"remove\",\"path\":\"/…/exclusiveMaximum\"},{\"op\":\"remove\",\"path\":\"/…/exclusiveMinimum\"}]"
			}
		}
	}
]
```
```json:patch resolves
"patches": [
	{
		"type": "unofficial",
		"resolves": [
			{
				"type": "defect",
				"description": "Removed deprecated boolean keywords(exclusiveMaximum,exclusiveMinimum)"
			}
		]
	}
]
```
```json releaseNotes-examples
"releaseNotes": {
	"type": "minor",
	"title": "入力検証の強化",
	"description": "空文字列の入力時に例外を投げるようにし、ログ出力の形式を整理した。",
	"timestamp": "2026-10-01T09:00:00+09:00",
	"tags": ["validation", "logging"],
	"resolves": [
		{
			"type": "defect",
			"id": "ISSUE-123",
			"name": "空文字列で NullPointerException が発生する",
			"source": {
				"name": "GitHub Issues",
				"url": "https://github.com/example/example-lib/issues/123"
			}
		},
		{
			"type": "enhancement",
			"id": "ISSUE-130",
			"name": "ログ形式の統一"
		}
	],
	"notes": [
		{
			"locale": "ja-JP",
			"text": {
				"contentType": "text/markdown",
				"content": "## 変更点\n- 空文字列を受け取ったとき、明示的な例外を投げる\n- ログを JSON 形式に統一\n\n## 移行時の注意\n既存のログ解析設定の見直しが必要です。"
			}
		},
		{
			"locale": "en-US",
			"text": {
				"contentType": "text/markdown",
				"content": "Added input validation and unified the log format."
			}
		}
	]
}
```