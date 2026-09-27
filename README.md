# Batch Poster Image Generation

Codex 技能，用於將大量教育海報提示詞建立佇列，分批生成視覺母版、記錄狀態、重試失敗項目，並進行 A2 放大、出血與排版前 QA。

目前版本：**v0.1.0**

## 使用

在 Codex 中提供提示詞目錄與輸出位置，並要求使用 `batch-poster-image-generation` 技能。預設每批 6 張，不需逐張確認；每張會建立穩定的 `P###-english-topic-YYYYMMDD` 檔名。

## A2 工作流

先以 GPT-Image-2.5 Sunburst 產生約 `2352×3328` 的乾淨視覺母版，再放大、加 3 mm 出血，最後在 Canva、PowerPoint、Affinity Publisher 或 SVG 中重新排入繁體中文。AI 母版本身不視為印刷完成稿。

詳細規則請見 [`SKILL.md`](SKILL.md) 與 [`references/manifest-schema.md`](references/manifest-schema.md)。
