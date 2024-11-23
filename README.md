//@version=5
indicator("Enhanced Heikin Ashi with Volume Analysis", overlay=true)

// --- Heikin Ashi Calculation ---
var float ha_open = na  // Persistent variable for Heikin Ashi open
ha_close = (open + high + low + close) / 4  // Heikin Ashi close is the average of the current bar
ha_open := na(ha_open[1]) ? (open + close) / 2 : (ha_open[1] + ha_close[1]) / 2  // Heikin Ashi open depends on the previous bar
ha_high = max(high, max(ha_open, ha_close))  // Heikin Ashi high
ha_low = min(low, min(ha_open, ha_close))  // Heikin Ashi low

// --- Heikin Ashi Candle Plotting ---
plotcandle(ha_open, ha_high, ha_low, ha_close, title="Heikin Ashi Candles", 
     color=(ha_close > ha_open ? color.green : color.red), 
     wickcolor=color.gray, 
     bordercolor=color.black)

// --- Volume Analysis ---
volume_average = ta.sma(volume, 20)  // 20-period simple moving average of volume
high_volume = volume > volume_average  // Detect if the current volume is above average

// --- Trend and Volume Conditions ---
bullish_trend = ha_close > ha_open  // Heikin Ashi candle is bullish
bearish_trend = ha_close < ha_open  // Heikin Ashi candle is bearish

buy_signal = bullish_trend and high_volume  // Buy signal: bullish candle with high volume
sell_signal = bearish_trend and high_volume  // Sell signal: bearish candle with high volume

// --- Plotting Signals ---
plotshape(buy_signal, title="Buy Signal", style=shape.labelup, location=location.belowbar, 
    color=color.green, text="Buy", size=size.small)
plotshape(sell_signal, title="Sell Signal", style=shape.labeldown, location=location.abovebar, 
    color=color.red, text="Sell", size=size.small)

// --- Volume Histogram ---
plot(volume, style=plot.style_histogram, color=(high_volume ? color.orange : color.gray), title="Volume")
hline(volume_average, "Volume Average", color=color.blue)

// --- Additional Information ---
bgcolor(buy_signal ? color.new(color.green, 90) : na, title="Buy Signal Background Highlight")
bgcolor(sell_signal ? color.new(color.red, 90) : na, title="Sell Signal Background Highlight")

// --- Alerts ---
alertcondition(buy_signal, title="Buy Alert", message="High Volume Bullish Signal Detected!")
alertcondition(sell_signal, title="Sell Alert", message="High Volume Bearish Signal Detected!")

// --- Debugging Lines (Optional for Developers) ---
// Display debug info in the data window
label.new(bar_index, high, str.tostring(volume, "Volume: #.##"), color=color.new(color.blue, 85), textcolor=color.white, style=label.style_circle)
