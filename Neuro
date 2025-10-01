// © stux

//@version=6
indicator("NeuroZone V1.0", overlay=true, max_lines_count=500, max_labels_count=500)

// ==========================
// Settings
// ==========================
showVolume = input.bool(true, "Show Volume & Heatmap")
volMAlen = input.int(20, "Volume MA Length")

showPercent = input.bool(true, "Show % Change & Stats")
lookbackForStats = input.int(100, "Lookback for Stats")

showFVG = input.bool(true, "Detect Fair Value Gaps (FVG)")
fvgLookback = input.int(50, "FVG Lookback Bars")

showOrderBlocks = input.bool(true, "Detect Order Blocks")
obLookback = input.int(50, "Order Block Lookback")
obMinRange = input.float(0.0005, "Min OB range (price units)")

showScore = input.bool(true, "Show Dynamic Score")
scoreWeightsVol = input.float(0.2, "Weight: Volume")
scoreWeightsFVG = input.float(0.25, "Weight: FVG")
scoreWeightsOB = input.float(0.25, "Weight: Order Blocks")
scoreWeightsTrend = input.float(0.3, "Weight: Trend")

mtf_tf = input.timeframe("D", "MTF timeframe for gap detection")
mtf_gap_size = input.float(0.5, "MTF gap size %")

showHeatmap = input.bool(true, "Show Price Heatmap")
heatmapOpacity = input.int(85, "Heatmap opacity (0-100)")

alertsOn = input.bool(true, "Enable Smart Alerts")
alertScoreThreshold = input.float(0.7, "Alert score threshold (0-1)")

showJournal = input.bool(true, "Show Trade Journal Table")
journalMaxRows = input.int(20, "Journal rows to show")

// ==========================
// Helpers
// ==========================
pctchange(src) => src == 0 ? 0.0 : (src - src[1]) / math.abs(src[1]) * 100.0

// ==========================
// Volume & Heatmap
// ==========================
vol = volume
volMA = ta.sma(vol, volMAlen)
plot(showVolume ? volMA : na, "Volume MA", style=plot.style_columns)

var color bgCol = na
if showVolume and showHeatmap
    normalizedVol = math.min(1, math.max(0, (vol - ta.lowest(vol, volMAlen)) / (ta.highest(vol, volMAlen) - ta.lowest(vol, volMAlen) + 1)))
    heatLevel = int(heatmapOpacity * normalizedVol)
    bgCol := color.new(color.orange, 100 - heatLevel)
bgcolor(showVolume and showHeatmap ? bgCol : na)

// ==========================
// Percent change & Stats
// ==========================
percent = showPercent ? pctchange(close) : na
plotchar(showPercent ? percent : na, title="% Change", location=location.top, char='%', size=size.tiny)

upCount = 0
for i = 0 to lookbackForStats - 1
    upCount += close[i] > open[i] ? 1 : 0
winRate = lookbackForStats > 0 ? upCount / lookbackForStats : na

// ==========================
// FVG Detection (fixed)
// ==========================
var float[] fvg_top = array.new_float()
var float[] fvg_bot = array.new_float()

if showFVG
    array.clear(fvg_top)
    array.clear(fvg_bot)
    for i = 2 to fvgLookback
        if low[i - 2] > high[i]
            array.push(fvg_top, low[i - 2])
            array.push(fvg_bot, high[i])
        if high[i - 2] < low[i]
            array.push(fvg_top, high[i - 2])
            array.push(fvg_bot, low[i])

// only draw boxes if we have elements
if array.size(fvg_top) > 0
    maxToDraw = math.min(10, array.size(fvg_top) - 1)
    for j = 0 to maxToDraw
        // use single-line call to avoid multiline parsing issues
        box.new(bar_index - j * 3, array.get(fvg_top, j), bar_index - j * 3 - 2, array.get(fvg_bot, j), bgcolor=color.new(color.purple, 90))



// ==========================
// Order Blocks Detection
// ==========================
var float[] ob_top = array.new_float()
var float[] ob_bot = array.new_float()
if showOrderBlocks
    array.clear(ob_top)
    array.clear(ob_bot)
    for i = 1 to obLookback
        if close[i] < open[i] and high[i] - low[i] > obMinRange and ta.crossover(close, high[i])
            array.push(ob_top, high[i])
            array.push(ob_bot, low[i])
        if close[i] > open[i] and high[i] - low[i] > obMinRange and ta.crossunder(close, low[i])
            array.push(ob_top, high[i])
            array.push(ob_bot, low[i])

for j = 0 to math.min(10, array.size(ob_top) - 1)
    box.new(bar_index - j * 2, array.get(ob_top, j), bar_index - j * 2 - 1, array.get(ob_bot, j), bgcolor=color.new(color.teal, 90))

// ==========================
// Trend & Score
// ==========================
emaFast = ta.ema(close, 9)
emaSlow = ta.ema(close, 21)
trend = emaFast > emaSlow ? 1 : -1
plot(emaFast, "EMA Fast")
plot(emaSlow, "EMA Slow")

volScore = math.min(1, math.max(0, (vol - ta.lowest(vol, volMAlen)) / (ta.highest(vol, volMAlen) - ta.lowest(vol, volMAlen) + 1)))
fvgScore = array.size(fvg_top) > 0 ? 1 : 0
obScore = array.size(ob_top) > 0 ? 1 : 0
trendScore = trend == 1 ? 1 : 0

score = math.round((scoreWeightsVol * volScore + scoreWeightsFVG * fvgScore + scoreWeightsOB * obScore + scoreWeightsTrend * trendScore) * 100) / 100.0
if showScore
    label.new(bar_index, high, "Score: " + str.tostring(score), style=label.style_label_right, color=score > 0.6 ? color.new(color.green, 30) : color.new(color.red, 50))

// ==========================
// MTF Gap Detection
// ==========================
[mtf_open, mtf_high, mtf_low, mtf_close] = request.security(syminfo.tickerid, mtf_tf, [open, high, low, close])
mtf_gap_pct = mtf_close == 0 ? 0 : (close - mtf_close) / math.abs(mtf_close) * 100
if math.abs(mtf_gap_pct) >= mtf_gap_size
    label.new(bar_index, low, "MTF GAP: " + str.tostring(mtf_gap_pct, format.percent), style=label.style_label_left, color=color.new(color.orange, 30))

// ==========================
// Buy / Sell Signals
// ==========================
buySignal = trend == 1 and vol > volMA * 1.3 and array.size(ob_top) > 0
sellSignal = trend == -1 and vol > volMA * 1.3 and array.size(ob_top) > 0
if buySignal
    label.new(bar_index, low, "BUY", style=label.style_label_up, color=color.green)
if sellSignal
    label.new(bar_index, high, "SELL", style=label.style_label_down, color=color.red)

// ==========================
// Alerts
// ==========================
if alertsOn and score >= alertScoreThreshold
    alert("Wolves Alert: Score=" + str.tostring(score), alert.freq_once_per_bar_close)
alertcondition(alertsOn and score >= alertScoreThreshold, "Wolves Smart Alert", "Wolves Alert Triggered")

// ==========================
// Trade Journal
// ==========================
var table journal = showJournal ? table.new(position.top_right, 3, journalMaxRows) : na
var string[] j_time = array.new_string()
var string[] j_side = array.new_string()
var float[] j_price = array.new_float()

if buySignal
    array.unshift(j_time, str.tostring(time, "yyyy-MM-dd HH:mm"))
    array.unshift(j_side, "BUY")
    array.unshift(j_price, close)
if sellSignal
    array.unshift(j_time, str.tostring(time, "yyyy-MM-dd HH:mm"))
    array.unshift(j_side, "SELL")
    array.unshift(j_price, close)

while array.size(j_time) > journalMaxRows
    array.pop(j_time)
    array.pop(j_side)
    array.pop(j_price)

if showJournal and not na(journal)
    table.clear(journal, 0, 0, 2, journalMaxRows - 1)
    table.cell(journal, 0, 0, "Time", text_color=color.white, bgcolor=color.gray)
    table.cell(journal, 1, 0, "Side", text_color=color.white, bgcolor=color.gray)
    table.cell(journal, 2, 0, "Price", text_color=color.white, bgcolor=color.gray)
    for i = 0 to array.size(j_time) - 1
        table.cell(journal, 0, i + 1, array.get(j_time, i))
        table.cell(journal, 1, i + 1, array.get(j_side, i))
        table.cell(journal, 2, i + 1, str.tostring(array.get(j_price, i)))

// ==========================
// Summary Label
// ==========================
var label summary = label.new(bar_index, na, "", xloc=xloc.bar_index, yloc=yloc.price)
label.set_xy(summary, bar_index, high)
label.set_text(summary, "Score: " + str.tostring(score) + "\nVolMA: " + str.tostring(volMA) + "\nWin%: " + str.tostring(math.round(winRate * 100)) + "%")
