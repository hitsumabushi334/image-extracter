# mcp_lambda_handler ライブラリ使用方法 - 詳細調査レポート

## 概要

このレポートは、AWS LabsのMCP（Model Context Protocol）Lambda ハンドラーライブラリについて詳細に調査し、その使用方法と実装手順をまとめたものです。

## 1. 基本概要と目的

### ライブラリの基本機能
`mcp_lambda_handler`は、AWS Lambda上で**MCP（Model Context Protocol）サーバー**を簡単に構築・デプロイするために設計されたPythonライブラリです。

- **基本機能**: AWS Lambda関数を、MCP仕様に準拠したHTTPエンドポイントとして機能させる
- **目的**: サーバーレス環境でスケーラブルなMCPサーバーを実行可能
- **MCPプロトコル**: LLM（大規模言語モデル）アプリケーションが外部ツールやデータソースと対話するためのオープンプロトコル

### AWS Lambda環境での利点
- サーバーのプロビジョニングや管理が不要
- 自動スケーリング機能
- 従量課金制による コスト効率性
- AWSのセキュリティ機能を活用可能

## 2. アーキテクチャと設計

### 主要なクラス構造
```
MCPLambdaHandler
├── __init__(name, version, ...)
├── handle_request(event, context)
└── tool() [デコレータ]
```

### 主要コンポーネント

#### MCPLambdaHandler クラス
- **役割**: ライブラリの中心となるクラス
- **機能**: MCPサーバーの定義、ツール登録、リクエスト処理

#### 主要メソッド
- `__init__(name, version, ...)`: MCPサーバーを初期化
- `handle_request(event, context)`: Lambdaのevent/contextオブジェクトを処理
- `tool()`: 関数をMCPツールとして登録するデコレータ

## 3. 実装手順と使用方法

### ステップ 1: ライブラリのインストール
```bash
pip install awslabs-mcp-lambda-handler
```

### ステップ 2: 基本的な実装
```python
from awslabs.mcp_lambda_handler import MCPLambdaHandler
from datetime import datetime, UTC

# MCPサーバーのインスタンス作成
mcp_server = MCPLambdaHandler(name="mcp-lambda-server-example", version="1.0.0")

# ツールとして関数を登録
@mcp_server.tool()
def get_current_time() -> str:
    """
    Returns the current time in UTC ISO 8601 format.
    """
    return datetime.now(UTC).isoformat()

# Lambdaエントリーポイント
def lambda_handler(event, context):
    """
    AWS Lambda handler function.
    """
    return mcp_server.handle_request(event, context)
```

### ステップ 3: AWS設定

#### Lambda関数の設定
1. コードと依存関係をzipファイルにパッケージング
2. Lambda関数にアップロード
3. ハンドラを `ファイル名.lambda_handler` に設定

#### IAMロールの設定
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        }
    ]
}
```

#### API Gateway統合（推奨）
- HTTP APIまたはREST APIを作成
- Lambda関数と統合
- 外部からのHTTPアクセスを有効化

## 4. リクエスト/レスポンス処理メカニズム

### 処理フロー
1. **クライアント** → API Gateway → Lambda関数
2. **Lambda**: eventオブジェクトを受信
3. **handle_request**: eventをMCPリクエストに変換
4. **ツール実行**: 登録された関数を呼び出し
5. **レスポンス**: 実行結果をMCP形式のJSONに整形
6. **返却**: API Gateway経由でクライアントに応答

### データフロー図
```
[クライアント] 
    ↓ HTTP Request
[API Gateway] 
    ↓ Lambda Event
[MCPLambdaHandler] 
    ↓ MCP Protocol
[登録済みツール] 
    ↓ 実行結果
[MCPレスポンス]
    ↓ HTTP Response
[クライアント]
```

## 5. エラーハンドリングとデバッグ

### エラーハンドリング戦略
```python
@mcp_server.tool()
def safe_operation(input_data: str) -> str:
    """
    安全な操作を実行する例
    """
    try:
        # 何らかの処理
        result = process_data(input_data)
        return result
    except ValueError as e:
        return f"入力エラー: {str(e)}"
    except Exception as e:
        return f"処理エラー: {str(e)}"
```

### デバッグ方法

#### 1. CloudWatch Logs
- Lambda実行ログの確認
- print文やloggingモジュールでのデバッグ出力
- エラースタックトレースの分析

#### 2. MCP Inspector
- MCP公式テストツール
- リクエスト/レスポンスの検証
- プロトコル準拠性の確認

#### 3. ローカルテスト
```python
# ローカルテスト用のeventオブジェクト作成
test_event = {
    "httpMethod": "POST",
    "body": json.dumps({
        "jsonrpc": "2.0",
        "method": "tools/call",
        "params": {
            "name": "get_current_time",
            "arguments": {}
        }
    })
}

result = lambda_handler(test_event, {})
print(json.dumps(result, indent=2))
```

## 6. 実装例とユースケース

### 例1: AWS S3ファイル操作ツール
```python
import boto3

s3_client = boto3.client('s3')

@mcp_server.tool()
def list_s3_objects(bucket_name: str, prefix: str = "") -> list:
    """
    S3バケット内のオブジェクト一覧を取得
    
    Args:
        bucket_name: バケット名
        prefix: プレフィックス（オプション）
    
    Returns:
        オブジェクトキーのリスト
    """
    try:
        response = s3_client.list_objects_v2(
            Bucket=bucket_name,
            Prefix=prefix
        )
        return [obj['Key'] for obj in response.get('Contents', [])]
    except Exception as e:
        return f"エラー: {str(e)}"
```

### 例2: データベース クエリツール
```python
import json

@mcp_server.tool()
def query_database(query: str, table_name: str) -> dict:
    """
    データベースクエリを実行
    
    Args:
        query: 実行するクエリ
        table_name: テーブル名
        
    Returns:
        クエリ結果
    """
    # セキュリティ: SQLインジェクション対策必須
    if not is_safe_query(query, table_name):
        return {"error": "安全でないクエリです"}
    
    # データベース接続・実行処理
    # ... 実装 ...
    
    return {"results": [], "count": 0}
```

## 7. セッション管理とステート管理

### DynamoDBセッション管理
ライブラリは複数のツール呼び出し間での状態維持をサポートします：

```python
# セッション対応の実装例
@mcp_server.tool()
def remember_context(user_input: str, session_id: str = None) -> str:
    """
    ユーザー入力を記憶し、コンテキストを維持
    """
    if session_id:
        # DynamoDBからセッションデータを取得
        session_data = get_session_data(session_id)
        context = session_data.get('context', [])
    else:
        context = []
    
    # 新しい入力を追加
    context.append(user_input)
    
    # セッションデータを更新
    if session_id:
        update_session_data(session_id, {'context': context})
    
    return f"記憶しました: {user_input}"
```

## 8. ベストプラクティスと注意点

### docstring の重要性
```python
@mcp_server.tool()
def good_tool_example(param1: str, param2: int = 10) -> dict:
    """
    ツールの明確な説明をここに記述
    
    Args:
        param1: パラメータ1の説明（必須）
        param2: パラメータ2の説明（オプション、デフォルト値: 10）
        
    Returns:
        処理結果の辞書形式データ
        
    Examples:
        入力例と期待される出力例
    """
    return {"result": f"処理完了: {param1}", "count": param2}
```

### セキュリティ考慮事項

#### 1. 認証・認可
```python
# API Gateway オーソライザーの使用推奨
# IAM認証やCognito認証の実装

def validate_authorization(event):
    """認証情報の検証"""
    auth_header = event.get('headers', {}).get('Authorization')
    if not auth_header:
        raise Exception('認証が必要です')
    # トークン検証ロジック
```

#### 2. 入力検証
```python
import re

@mcp_server.tool()
def secure_tool(user_input: str) -> str:
    """
    入力を安全に処理するツールの例
    """
    # 入力値の検証
    if not re.match(r'^[a-zA-Z0-9\s]+$', user_input):
        return "エラー: 無効な文字が含まれています"
    
    if len(user_input) > 1000:
        return "エラー: 入力が長すぎます"
    
    # 処理実行
    return f"処理完了: {user_input}"
```

### パフォーマンス最適化

#### 1. コールドスタート対策
```python
# グローバルスコープでの初期化
s3_client = boto3.client('s3')  # Lambda コンテナ再利用時に初期化を省略

def lambda_handler(event, context):
    # ハンドラ内では既存のクライアントを使用
    return mcp_server.handle_request(event, context)
```

#### 2. メモリ使用量最適化
```python
import gc

@mcp_server.tool()
def memory_efficient_tool(large_data: list) -> str:
    """
    メモリ効率を考慮したツール実装
    """
    try:
        # 大きなデータの処理
        result = process_large_data(large_data)
        
        # 明示的なガベージコレクション
        del large_data
        gc.collect()
        
        return result
    except MemoryError:
        return "メモリ不足エラー: データサイズを小さくしてください"
```

## 9. トラブルシューティング

### よくある問題と解決策

#### 問題1: Lambda タイムアウト
```python
# 解決策: 処理時間の監視と早期終了
import time

def lambda_handler(event, context):
    start_time = time.time()
    remaining_time = context.get_remaining_time_in_millis() / 1000
    
    if remaining_time < 10:  # 10秒未満の場合は処理をスキップ
        return {"error": "処理時間が不足しています"}
    
    return mcp_server.handle_request(event, context)
```

#### 問題2: MCPプロトコル エラー
```python
# 解決策: リクエスト形式の検証
def validate_mcp_request(event):
    """MCPリクエストの妥当性検証"""
    try:
        body = json.loads(event.get('body', '{}'))
        required_fields = ['jsonrpc', 'method']
        
        for field in required_fields:
            if field not in body:
                raise ValueError(f"必須フィールドが不足: {field}")
                
        return True
    except (json.JSONDecodeError, ValueError) as e:
        print(f"MCP リクエスト検証エラー: {e}")
        return False
```

## 10. 参考リンク・リソース

- [AWS MCP Lambda Handler GitHub Repository](https://github.com/awslabs/mcp/tree/main/src/mcp-lambda-handler)
- [Model Context Protocol公式仕様](https://modelcontextprotocol.io/)
- [AWS Lambda開発者ガイド](https://docs.aws.amazon.com/lambda/latest/dg/)
- [MCP Inspector ツール](https://github.com/modelcontextprotocol/inspector)

## まとめ

`mcp_lambda_handler`ライブラリは、AWS Lambdaを使用してMCPサーバーを構築するための強力で使いやすいツールです。適切な設計パターンとセキュリティ考慮事項を組み込むことで、スケーラブルで信頼性の高いMCPサーバーを効率的に開発することができます。

実装時は、docstringの詳細記述、入力検証、エラーハンドリング、セキュリティ対策を十分に検討し、本番環境での運用に備えることが重要です。