TABLE study_hours as "時間", topics as "トピック"
FROM #daily
WHERE study_hours > 0
SORT file.day DESC
LIMIT 7

// ここで合計時間を計算
let pages = dv.pages("#daily");
let total = 0;
for (let p of pages) {
    if (p.study_hours) {
        total += p.study_hours;
    }
}
dv.header(2, "🔥 現在の総学習時間: " + total + " 時間");