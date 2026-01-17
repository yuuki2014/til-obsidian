```dataview
TABLE study_hours as "時間", topics as "トピック"
FROM #daily
WHERE study_hours > 0
SORT file.day DESC
LIMIT 7
```
```dataviewjs
// ここで合計時間を計算
let pages = dv.pages("#daily");
let total = 0;
for (let p of pages) {
    if (p.study_hours) {
        total += p.study_hours;
    }
}
dv.header(2, "🔥 現在の総学習時間: " + total + " 時間");
```

```dataviewjs
dv.span("**💻 学習ヒートマップ**")

const calendarData = {
    year: 2026,  // ← 翌年はここを変える
    entries: [],
}

// デイリーノートからデータを取得
for (let page of dv.pages('"00_Inbox" or "Daily"').where(p => p.study_hours)) {
    calendarData.entries.push({
        date: page.file.name, // デイリーノートのファイル名が "YYYY-MM-DD" である前提
        intensity: page.study_hours, // 色の濃さを勉強時間にする
        content: await dv.span(page.study_hours + "h"), // マウスホバーで時間を表示
    })
}

renderHeatmapCalendar(this.container, calendarData)
```
