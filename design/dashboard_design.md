 (cd "$(git rev-parse --show-toplevel)" && git apply --3way <<'EOF' 
diff --git a/design/dashboard_design.md b/design/dashboard_design.md
index 31d3b8f9d0352b07323b51f9bdddd11e2d8b3e9e..df532b4c1cb8acdbb1021fdc1c8358337c317600 100644
--- a/design/dashboard_design.md
+++ b/design/dashboard_design.md
@@ -1 +1,95 @@
-Power BI dashboard design description
+# プロジェクト一覧 Power BIダッシュボード設計
+
+## 1. ダッシュボードの目的
+既存の「プロジェクト一覧」画面をPower BIで再現し、以下を1ページで判断できるようにする。
+- 全体の案件ボリューム（件数）
+- 進捗・予算・遅延の健全性
+- リスクが高い案件の早期特定
+
+## 2. ページ構成（16:9 / 1366x768想定）
+
+### 上段（KPIカード 6個）
+1. 総プロジェクト数
+2. 進行中プロジェクト数
+3. 完了率
+4. 予算消化率
+5. 遅延プロジェクト数
+6. 高リスクプロジェクト数
+
+### 中段（分析チャート 3個）
+- **左**: ステータス内訳（ドーナツ）
+  - 凡例: Status
+  - 値: Project Count
+- **中央**: 予算計画 vs 実績（集合縦棒）
+  - 軸: Department
+  - 値: Planned Cost / Actual Cost
+- **右**: 進捗×予算消化（散布図）
+  - X: Progress %
+  - Y: Budget Burn Rate %
+  - サイズ: Planned Cost
+  - 凡例: RiskLevel
+
+### 下段
+- **左**: 月次完了件数推移（折れ線）
+  - 軸: YearMonth
+  - 値: Completed Projects
+- **右**: プロジェクト詳細テーブル
+  - ProjectName, Department, ProjectManager, Status, RiskLevel
+  - Progress %, Planned Cost, Actual Cost, Cost Variance, Delay Days
+
+### 右ペイン（スライサー）
+- Department
+- ProjectManager
+- Status
+- RiskLevel
+- Date（相対日付）
+
+## 3. インタラクション
+- スライサーはページ全ビジュアルに適用。
+- チャート選択時は他チャート + テーブルをクロスフィルタ。
+- KPIカードはフィルタ起点にしない（閲覧専用）。
+- テーブルは `Delay Days` 降順を既定ソート。
+
+## 4. 条件付き書式ルール
+
+### Progress %
+- `>= 80%`: 緑
+- `>= 50% and < 80%`: 黄
+- `< 50%`: 赤
+
+### Cost Variance（Actual - Planned）
+- `<= 0`: 緑（予算内）
+- `> 0 and <= 10% of Planned`: 黄
+- `> 10% of Planned`: 赤
+
+### Delay Days
+- `<= 0`: 緑
+- `1 - 14`: 黄
+- `>= 15`: 赤
+
+### RiskLevel
+- Low: 緑
+- Medium: 黄
+- High: 赤
+
+## 5. 想定データ項目（最低限）
+- ProjectID
+- ProjectName
+- Department
+- ProjectManager
+- Status
+- RiskLevel
+- StartDate
+- PlannedEndDate
+- EndDate
+- PlannedCost
+- ActualCost
+- ProgressPct（0〜1）
+
+## 6. 実装順序
+1. モデル接続（型/リレーション確認）
+2. DAXメジャー作成
+3. レイアウト配置
+4. 条件付き書式設定
+5. テーマ適用
+6. パフォーマンスと操作性の最終調整
 
EOF
)
