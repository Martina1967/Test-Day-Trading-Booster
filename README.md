# Test-Day-Trading-Booster
Indikator von DGT 
// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © Jojo67

//@version=5

// Functions

f_drawOnlyLineX(_x1, _y1, _x2, _y2, _xloc, _extend, _color, _style, _width) =>
    line.new(_x1, _y1, _x2, _y2, _xloc, _extend, _color, _style, _width)

f_drawLineX(_x1, _y1, _x2, _y2, _xloc, _extend, _color, _style, _width) =>
    var id = line.new(_x1, _y1, _x2, _y2, _xloc, _extend, _color, _style, _width)

    if _y1 > 0 and _y2 > 0
        line.set_xy1(id, _x1, _y1)
        line.set_xy2(id, _x2, _y2)
        line.set_color(id, _color)
    else
        line.set_xy1(id, _x1, close)
        line.set_xy2(id, _x2, close)
        line.set_color(id, #00000000)

f_drawOnlyLabelX(_x, _y, _text, _xloc, _yloc, _color, _style, _textcolor, _size, _textalign, _tooltip) =>
    label.new(_x, _y, _text, _xloc, _yloc, _color, _style, _textcolor, _size, _textalign, _tooltip)

f_drawLabelX(_x, _y, _text, _xloc, _yloc, _color, _style, _textcolor, _size, _textalign, _tooltip) =>
    var id = label.new(_x, _y, _text, _xloc, _yloc, _color, _style, _textcolor, _size, _textalign, _tooltip)
    label.set_text(id, _text)
    label.set_tooltip(id, _tooltip)
    
    if _y > 0
        label.set_xy(id, _x, _y)
        label.set_textcolor(id, _textcolor)
    else
        label.set_xy(id, _x, close)
        label.set_textcolor(id, #00000000)

f_calculatePreviousRange(_htf) =>
    var htf_h  = 0., var htf_l  = 0., var htf_hx = 0., var htf_lx = 0.

    if ta.change(time(_htf))
        htf_hx := htf_h, htf_h  := high
        htf_lx := htf_l, htf_l  := low
        true
    else
        htf_h := math.max(high, htf_h)
        htf_l := math.min(low , htf_l)
        true

    [htf_hx, htf_lx]

f_getTradedVolume(_len, _calc, _offset) =>
    if _calc
        vol   = 0.
        for x = 0 to _len - 1
            vol += volume[_offset + x]
        vol

f_getMA(_source, _length, _type) => 
    switch _type
        "SMA"  => ta.sma (_source, _length)
        "EMA"  => ta.ema (_source, _length)
        "HMA"  => ta.hma (_source, _length)
        "WMA"  => ta.wma (_source, _length)

f_getATR(_length, _htf) =>
    request.security(syminfo.tickerid, _htf, ta.atr(_length))


strategy('Test Day Trading Booster ', 'Test Day Trading Booster ☼☾', true, max_lines_count=500, max_labels_count=500)

// ---------------------------------------------------------------------------------------------- //
// Input Declarations

group_fx = "Forex Markets"

ttip_channel = 'Asian Session Opening Channel is the range between Tokyo Open and Hong Kong Open\n' +
               'Europien Session Opening Channel is the range between Frankfurt Open and London Open\n\n' +
               'Channel plotting starts from Tokyo or Frankfurt open, based on the option selected, and extends till the day session end'

openingChannel = input.string('None' , 'Openning Channel'  , options = ['Asian Session', 'European Session', 'Both', 'None'], inline='ss', group = group_fx, tooltip = ttip_channel)
cChannel       = input.color(color.gray, '', inline='ss', group = group_fx)

shSydney  = input.string('None' , 'Sydney Session '  , options = ['Open', 'Range', '5m Opening Range', '15m Opening Range', 'None'], inline='SY' , group = group_fx)
cSydney   = input.color(color.navy, '', inline='SY', group = group_fx)
bgSydney  = true//input.bool(true, 'Range Fill', inline='SY', group = group_fx)
i_sydneySummerTime  = input.bool(true, 'DST', inline='SY' , group = group_fx, tooltip = 'Daylight saving time (DST)\n *DST Start : First Sunday in October at 2:00\n *DST End : First Sunday in April at 3:00')

shTokyo  = input.string('None' , 'Tokyo Session  '  , options = ['Open', 'Range', '5m Opening Range', '15m Opening Range', 'None'], inline='TK' , group = group_fx)
cTokyo   = input.color(color.fuchsia, '', inline='TK', group = group_fx)
bgTokyo  = true//input.bool(true, 'Range Fill', inline='TK', group = group_fx)

shFrankfurt  = input.string('5m Opening Range' , 'Frankfurt Session'  , options = ['Open', 'Range', '5m Opening Range', '15m Opening Range', 'None'], inline='FR' , group = group_fx)
cFrankfurt   = input.color(color.yellow, '', inline='FR', group = group_fx)
bgFrankfurt  = true//input.bool(true, 'Range Fill', inline='FR', group = group_fx)
i_frankfurtSummerTime  = input.bool(false , 'DST', inline='FR', group = group_fx, tooltip = 'Daylight saving time (DST)\n *DST Start : Last Sunday in March at 1:00 UTC\n *DST End : Last Sunday in October at 1:00 UTC')

shLondon  = input.string('5m Opening Range' , 'London Session '  , options = ['Open', 'Range', '5m Opening Range', '15m Opening Range', 'None'], inline='LN' , group = group_fx)
cLondon   = input.color(color.blue, '', inline='LN', group = group_fx)
bgLondon  = true//input.bool(true, 'Range Fill', inline='LN', group = group_fx)
i_londonSummerTime  = input.bool(false , 'DST', inline='LN', group = group_fx, tooltip = 'Daylight saving time (DST)\n *DST Start : Last Sunday in March at 1:00 UTC\n *DST End : Last Sunday in October at 1:00 UTC')

shNYork  = input.string('15m Opening Range' , 'New York Session'  , options = ['Open', 'Range', '5m Opening Range', '15m Opening Range', 'None'], inline='NY' , group = group_fx)
cNYork   = input.color(color.green, '', inline='NY', group = group_fx)
bgNYork  = true//input.bool(true, 'Range Fill', inline='NY', group = group_fx)
i_newyorkSummerTime = input.bool(false , 'DST', inline='NY', group = group_fx, tooltip = 'Daylight saving time (DST)\n *DST Start : Second Sunday in March at 2:00\n *DST End : First Sunday in November at 2:00')

i_forex       = input.bool(true , 'Global Forex Market Sessions Tabular View'  , group = group_fx, tooltip = 'Displays Major Fortex Markets\n - Date and Time,\n - Sessions Opening/Closing Countdown Timer and\n - Market Status')
hideIfNot     = input.bool(true , 'Hide if not Forex Market Instrument'  , group = group_fx)
i_textSize2   = input.string("Small", "  Table Text Size", options = [ "Tiny", "Small", "Normal"], inline='STAT',group = group_fx)
textSize2           = i_textSize2 == "Small" ? size.small : i_textSize2 == "Normal" ? size.normal : size.tiny
statPos   = input.string('Top Right', '', options=['Top Left', 'Top Center', 'Top Right', 'Middle Right', 'Bottom Left', 'Bottom Center'], inline='STAT', group=group_fx) 

ttip_events   = 'A graphical presentation* of typically traded volume** and various forext markets oppening/clossing events***\n\n' +
                 '*   Will be presented on 1H or lower minute timeframes\n' +
                 '**  According to the two most popular currencies traded, U.S. dollar (USD) and the Euro (EUR). Trading volume may slightly vary for other currencies, especially Asian currencies\n' +
                 '*** Reminder, Daylight saving time is not taken into consideration so there may be slight differences'

forexEvents       = input.bool(true , 'Global Forex Market Events'  , group = group_fx, tooltip = ttip_events)
oscVerticalOffset = input.int(0, "  Vertical Offset", minval = -3, maxval = 10, inline='evt', group = group_fx) / 10
oscHight          = 11 - input.int(7, 'Hight' , minval = 1, maxval = 10 ,inline='evt', group = group_fx)

group_htf = 'Higher Timeframe Open'

dayOpen      = input.string('5m Opening Range' , 'Day Open     '  , options = ['Open', '5m Opening Range', '15m Opening Range', 'First Hour Opening Range', 'None'], inline='DO' , group = group_htf)
dayColor     = input.color(color.aqua, '', inline = 'DO', group = group_htf)
dayRefLevels = input.string('Average True Range (ATR)', 'Day Referance Levels', options = ['Average True Range (ATR)', 'Previous Day Range (PH - PL)', 'None'], inline = 'REFL', group = group_htf)
dayRefColor  = input.color(color.orange, '', inline = 'REFL', group = group_htf)
dayRefLevel1 = input.float(.5, '  Referance Level #1', minval=0, step=0.1, inline='Ref', group=group_htf)
dayRefLevel2 = input.float(1., '#2', minval=0, step=0.1, inline='Ref', group=group_htf)
dayATRLength = input.int(5, '  ATR Referance Levels : ATR Length', minval=1, inline='ATR', group=group_htf)

weekOpen   = input.bool(false , 'Week Open'  , inline='WO' , group = group_htf)
weekColor  = input.color(color.aqua, '', inline = 'WO', group = group_htf)
monthOpen  = input.bool(false , 'Month Open'  , inline='WO' , group = group_htf)
monthColor = input.color(color.aqua, '', inline = 'WO', group = group_htf)

group_indicators = 'Indicators'

ttip_vwap  = 'Volume Weighted Average Price (VWAP) is a technical analysis tool used to measure the average price weighted by volume. VWAP is typically used with intraday charts as a way to determine the general direction of intraday prices, that is to identify market trend'
ttip_vwapAnchor = 'Auto input mode : the starting point of the VWAP calculation depends on the timeframe on the chart:\n' +
                  ' - "Day" on intraday timeframes up to 15 Mins\n' +
                  ' - "Week" on intraday timeframes higher than 15 Mins\n' +
                  ' - "Month" on the 1D timeframe\n' +
                  ' - "Quarter" on the 1W timeframe\n' +
                  ' - "Year" on the 1M timeframe\n\n' +
                  'Interactive input mode : enables selecting the starting point of the VWAP calculation by simply dragging the vertical line* on the chart\n  *the vertical line appears when any of the indicators plotted components are clicked\n\n' +
                  'Manuel input mode (other options) : enables to specify a fixed anchor period regardless of the chart timeframe'

isVwap     = input.bool(false, 'Volume Weighted Average Price (VWAP) ----------|', group=group_indicators, tooltip = ttip_vwap)
htf_tf     = input.string('Auto', 'Anchor Timeframe', options=['Auto', 'Interactive', '15 Minutes', '1 Hour', '4 Hours', 'Daily', 'Weekly', 'Monthly', 'Quarterly', 'Yearly'], inline = 'vwap', group=group_indicators, tooltip = ttip_vwapAnchor)
vwapAnchor = htf_tf == '15 Minutes' ? '15' : htf_tf == '1 Hour' ? '60' : htf_tf == '4 Hours' ? '240' :  htf_tf == 'Daily' ? 'D' : htf_tf == 'Weekly' ? 'W' : htf_tf == 'Monthly' ? 'M' : htf_tf == 'Quarterly' ? '3M' : htf_tf == 'Yearly' ? '12M' : 
          timeframe.isintraday and (timeframe.period == '1'  or timeframe.period == '3'  or timeframe.period == '5'  or timeframe.period == '15') ? 'D' : 
          timeframe.isintraday and (timeframe.period == '30' or timeframe.period == '45' or timeframe.period == '60' or timeframe.period == '120' or timeframe.period == '180' or timeframe.period == '240') ? 'W' : 
          timeframe.isdaily ? 'M' : timeframe.isweekly ? '3M' : timeframe.ismonthly ? '12M' : '3M'

vwapSource = input.source(hlc3, 'Source', inline = 'vwap', group=group_indicators)
startDate  = input.time(timestamp("29 Nov 2022"), "Interactive Anchor", inline = 'inn', group=group_indicators)
vwapBand1  = input.bool(true,"Bands #1", inline = 'B1', group=group_indicators)
stdevMult1 = input.float(1, "", step = .1, inline = 'B1', group=group_indicators)
vwapBand2  = input.bool(false,"#2", inline = 'B1', group=group_indicators)
stdevMult2 = input.float(2, "", step = .1, inline = 'B1', group=group_indicators)
//vwapBand3  = input.bool(false,"Bands stDev #3", inline = 'B3', group=group_indicators)
//stdevMult3 = input.float(3, "", inline = 'B3', group=group_indicators)

tooltip_pvt = 'The Pivot Points High Low indicator is used to determine and anticipate potential changes in market price and reversals. By analyzing price changes and reversals, a trader has more of an ability to determine and predict price patterns and general price trends\n\n' +
              'This custom interpretation of Pivot Points High Low indicator; \n' + 
              ' - beside calculating Pivot Highs/Lows based on the given length it also calculates and detects short term Pivot Highs/Lows\n' +
              ' - calculates price changes, traded volume etc between any two Pivot High/Low'

dispPVT   = input.bool(true , 'Pivot Points High Low  ---------------------------|', group=group_indicators, tooltip=tooltip_pvt)
pvtLength = input.int(20, "  Left/Right Length", minval=1, group=group_indicators)
pvtText   = input.string('Small', "  Label Text Size", options=['Tiny', 'Small', 'Normal'], group=group_indicators)
pvtTextSize = pvtText == 'Small' ? size.small : pvtText == 'Normal' ? size.normal : size.tiny
pvtPrice  = input(true, "Price", inline = 'Levels', group=group_indicators)
pvtChange = input(false, "Price Change", inline = 'Levels', group=group_indicators)
pvtVolume = input(false, "Cumulative Volume", inline = 'Levels', group=group_indicators)

ttip_ma    = 'A Moving Average is a good way to gauge momentum as well as to confirm trends, and define areas of support and resistance.'

maDisplay  = input.bool(false , 'Moving Average  ----------------------------------|', group=group_indicators, tooltip=ttip_ma)
maType     = input.string('EMA', '', options=['SMA', 'EMA', 'WMA', 'HMA'], inline = 'MA', group = group_indicators)
maSource   = input.source(close,    "",  inline = 'MA', group = group_indicators)
maLength   = input.int(50,    "", minval = 1, inline = 'MA', group = group_indicators)
maColor    = input.color(color.orange, '', inline = 'MA', group = group_indicators)
maType2     = input.string('EMA', '', options=['SMA', 'EMA', 'WMA', 'HMA'], inline = 'MA2', group = group_indicators)
maSource2   = input.source(close,    "",  inline = 'MA2', group = group_indicators)
maLength2   = input.int(200,    "", minval = 1, inline = 'MA2', group = group_indicators)
maColor2    = input.color(color.orange, '', inline = 'MA2', group = group_indicators)

// ---------------------------------------------------------------------------------------------- //
// Variable Declarations

var a_majorCity     = array.new_string()
var a_utcTimeOffset = array.new_float()
var a_utcCity       = array.new_string()

var a_forexMarket   = array.new_string()
var a_forexOpenH    = array.new_float()
var a_forexCloseH   = array.new_float()

var a_forexEvents   = array.new_string()
var a_tradingVolume = array.new_float()

dayChange    = timeframe.change('D')
weekChange   = timeframe.change('W')
monthChange  = timeframe.change('M')
forex_n_cdf  = syminfo.type == 'forex' or syminfo.type == 'cfd'
tfMultiplier = timeframe.multiplier

if barstate.isfirst 
    // Forex Markets and UTC Zones
    array.push(a_majorCity, 'NEW YORK' ), array.push(a_utcTimeOffset, -5), array.push(a_utcCity, '(UTC-05:00) NEW YORK' )
    array.push(a_majorCity, 'LONDON'   ), array.push(a_utcTimeOffset, 0 ), array.push(a_utcCity, '(UTC+00:00) LONDON'   )
    array.push(a_majorCity, 'FRANKFURT'), array.push(a_utcTimeOffset, 1 ), array.push(a_utcCity, '(UTC+01:00) FRANKFURT')
    array.push(a_majorCity, 'TOKYO'    ), array.push(a_utcTimeOffset, 9 ), array.push(a_utcCity, '(UTC+09:00) TOKYO'    )
    array.push(a_majorCity, 'SYDNEY'   ), array.push(a_utcTimeOffset, 10), array.push(a_utcCity, '(UTC+10:00) SYDNEY'   )

    // Forex Market Open/Close Hours
    array.push(a_forexMarket, '(UTC+10:00) SYDNEY'   ), array.push(a_forexOpenH, 07), array.push(a_forexCloseH, 16)
    array.push(a_forexMarket, '(UTC+09:00) TOKYO'    ), array.push(a_forexOpenH, 09), array.push(a_forexCloseH, 18)
    array.push(a_forexMarket, '(UTC+01:00) FRANKFURT'), array.push(a_forexOpenH, 08), array.push(a_forexCloseH, 16)
    array.push(a_forexMarket, '(UTC+00:00) LONDON'   ), array.push(a_forexOpenH, 08), array.push(a_forexCloseH, 17)
    array.push(a_forexMarket, '(UTC-05:00) NEW YORK' ), array.push(a_forexOpenH, 08), array.push(a_forexCloseH, 17)

    // Tyipically Traded Volume and Forex Market Events 
    array.push(a_tradingVolume, 1   ) , array.push(a_forexEvents, 'Lowest \n - New York, Chicago and Troronto Closed' ) //00
    array.push(a_tradingVolume, 1   ) , array.push(a_forexEvents, 'Lowest' ) //01
    array.push(a_tradingVolume, 1.7 ) , array.push(a_forexEvents, 'Low \n - Tokyo Open' ) //02
    array.push(a_tradingVolume, 2.8 ) , array.push(a_forexEvents, 'Low and Slightly Increasing \n - Hong Kong, Shanghai and Singapore Open' ) //03
    array.push(a_tradingVolume, 3   ) , array.push(a_forexEvents, 'Low' ) //04
    array.push(a_tradingVolume, 2   ) , array.push(a_forexEvents, 'Low and Decreasing' ) //05
    array.push(a_tradingVolume, 1.8 ) , array.push(a_forexEvents, 'Low \n - Dubai Open, Wellington Closed' ) //06
    array.push(a_tradingVolume, 2.3 ) , array.push(a_forexEvents, 'Low and Increasing\n - Moscow Open' ) //07
    array.push(a_tradingVolume, 3   ) , array.push(a_forexEvents, 'Low and Increasing\n - Johannesburg Open, Sydney Closed' ) //08
    array.push(a_tradingVolume, 4.5 ) , array.push(a_forexEvents, 'Medium \n - Frankfurt Open' ) // 09
    array.push(a_tradingVolume, 5.7 ) , array.push(a_forexEvents, 'High and Increasing \n - London, Zurich and Paris Open' ) // 10
    array.push(a_tradingVolume, 6   ) , array.push(a_forexEvents, 'High \n - Tokyo, Hong Kong, Shanghai and Singapore Closed' ) // 11
    array.push(a_tradingVolume, 5.3 ) , array.push(a_forexEvents, 'High \n - Dubai Closed' ) // 12
    array.push(a_tradingVolume, 4.8 ) , array.push(a_forexEvents, 'High \n - India closes in 30 mins' ) // 13
    array.push(a_tradingVolume, 5.1 ) , array.push(a_forexEvents, 'High and Increasing \n - India Closed (30 mins ago)' ) // 14
    array.push(a_tradingVolume, 6.8 ) , array.push(a_forexEvents, 'Higher \n - New York and Toronto Open' ) // 15
    array.push(a_tradingVolume, 8.1 ) , array.push(a_forexEvents, 'Higher \n - Chicago Open, Frankfurt Closed' ) // 16
    array.push(a_tradingVolume, 9   ) , array.push(a_forexEvents, 'Highest \n - Johannesburg Closed' ) // 17
    array.push(a_tradingVolume, 7.8 ) , array.push(a_forexEvents, 'Higher and Decreasing \n - London and Moscow Closed' ) // 18
    array.push(a_tradingVolume, 5.3 ) , array.push(a_forexEvents, 'Medium and Decreasing' ) // 19
    array.push(a_tradingVolume, 3.1 ) , array.push(a_forexEvents, 'Low and Decreasing' ) // 20
    array.push(a_tradingVolume, 2   ) , array.push(a_forexEvents, 'Low and Decreasing \n - Wellington Open' ) // 21
    array.push(a_tradingVolume, 1.3 ) , array.push(a_forexEvents, 'Low and Decreasing' ) // 22
    array.push(a_tradingVolume, 1   ) , array.push(a_forexEvents, 'Lowest \n - Sydney Open' ) // 23
    array.push(a_tradingVolume, 1 ) // Dummy required for Trading Volume display

// ---------------------------------------------------------------------------------------------- //
// Forex Market Sessions Tabular View ----------------------------------------------------------- //

f_whatIsTheTime(_utc, _dst) =>
    TZ = 'Etc/UTC'
    DST = _dst ? 1 : 0
    utcTime = (array.get(a_utcTimeOffset, array.indexof(a_utcCity, _utc)) + DST) * 3600000 + timenow
    [math.floor(utcTime / 3600000) % 24, math.floor(utcTime / 60000) % 60, math.floor(utcTime / 1000) % 60, dayofmonth(int(utcTime), TZ), month(int(utcTime), TZ), year(int(utcTime), TZ), dayofweek(int(utcTime), TZ)]

f_digitalDisplay(_utc, _marketDetails, _dst) =>

    [h, m, s, D, M, Y, A] = f_whatIsTheTime(_utc, _dst)

    ht = h < 10 ? '0' + str.tostring(h) : str.tostring(h)
    mt = m < 10 ? '0' + str.tostring(m) : str.tostring(m)
    st = s < 10 ? '0' + str.tostring(s) : str.tostring(s)
    Dt = D < 10 ? '0' + str.tostring(D) : str.tostring(D)
    Mt = M < 10 ? '0' + str.tostring(M) : str.tostring(M)
    Yt = str.tostring(Y)
    dateTime = Dt + '/' + Mt + '/' + Yt + '-' + ht + ':' + mt + ':' + st

    if _marketDetails
        if A != 1 and A != 7
            fxO = array.get(a_forexOpenH , array.indexof(a_forexMarket, _utc))
            fxC = array.get(a_forexCloseH, array.indexof(a_forexMarket, _utc))

            market = if h >= fxO and h < fxC
                hc = fxC - h - 1
                mc = 60 - m - 1
                sc = 60 - s
                sct = sc < 10 ? '0' + str.tostring(sc) : str.tostring(sc)
                mct = mc < 10 ? '0' + str.tostring(mc) : str.tostring(mc)
                hct = hc < 10 ? '0' + str.tostring(hc) : str.tostring(hc)
                closes = hct + ':' + mct + ':' + sct

                if hc == 0
                    sc % 2 ? dateTime + ' 🟢 Closes in ' + closes : dateTime + ' 🔴 Closes in ' + closes
                else
                    dateTime + ' 🟢 Closes in ' + closes
            else
                ho = if h < fxO
                    fxO - h - 1
                else
                    24 - h + fxO - 1
                mo = 60 - m - 1
                so = 60 - s
                sot = so < 10 ? '0' + str.tostring(so) : str.tostring(so)
                mot = mo < 10 ? '0' + str.tostring(mo) : str.tostring(mo)
                hot = ho < 10 ? '0' + str.tostring(ho) : str.tostring(ho)
                opens = hot + ':' + mot + ':' + sot

                if h >= fxC and A == 6
                    dateTime + ' 🟠 Weekend'
                else
                    if ho == 0
                        so % 2 ? dateTime + ' 🔴 Opens in ' + opens : dateTime + ' 🟢 Opens in ' + opens
                    else
                        dateTime + ' 🔴 Opens in ' + opens
            market
        else
            dateTime + ' 🟠 Weekend'
    else
        dateTime

statPosition = switch statPos
    'Top Left'      => position.top_left
    'Top Center'    => position.top_center
    'Top Right'     => position.top_right
    'Middle Right'  => position.middle_right
    'Bottom Left'   => position.bottom_left
    'Bottom Center' => position.bottom_center

var table clock = table.new(statPosition, 3, 5, border_width = 3)

hide = hideIfNot ? forex_n_cdf ? true : false : true

if barstate.islast and i_forex and hide
    market = f_digitalDisplay('(UTC+10:00) SYDNEY', true, i_sydneySummerTime )
    marketColor = str.contains(market, 'Closes') ? #26a69a : #ef5350
    table.cell(clock, 0, 0, "█", text_size = textSize2, text_color = cSydney)
    table.cell(clock, 1, 0, array.get(a_majorCity, array.indexof(a_utcCity, '(UTC+10:00) SYDNEY'))    , text_color = color.blue, bgcolor = color.new(color.blue,  75), text_halign = text.align_left, text_size = textSize2)
    table.cell(clock, 2, 0, market                                                                    , text_color = marketColor , bgcolor =   color.new(marketColor, 75), text_halign = text.align_left, text_size = textSize2)

    market := f_digitalDisplay('(UTC+09:00) TOKYO', true , false)
    marketColor := str.contains(market, 'Closes') ? #26a69a : #ef5350
    table.cell(clock, 0, 1, "█", text_size = textSize2, text_color = cTokyo)
    table.cell(clock, 1, 1, array.get(a_majorCity, array.indexof(a_utcCity, '(UTC+09:00) TOKYO'))     , text_color = color.blue, bgcolor = color.new(color.blue,  75), text_halign = text.align_left, text_size = textSize2)
    table.cell(clock, 2, 1, market                                                                    , text_color = marketColor , bgcolor =   color.new(marketColor, 75), text_halign = text.align_left, text_size = textSize2)

    market := f_digitalDisplay('(UTC+01:00) FRANKFURT', true , i_frankfurtSummerTime)
    marketColor := str.contains(market, 'Closes') ? #26a69a : #ef5350
    table.cell(clock, 0, 2, "█", text_size = textSize2, text_color = cFrankfurt)
    table.cell(clock, 1, 2, array.get(a_majorCity, array.indexof(a_utcCity, '(UTC+01:00) FRANKFURT')) , text_color = color.blue, bgcolor = color.new(color.blue,  75), text_halign = text.align_left, text_size = textSize2)
    table.cell(clock, 2, 2, market                                                                    , text_color = marketColor , bgcolor =   color.new(marketColor, 75), text_halign = text.align_left, text_size = textSize2)

    market := f_digitalDisplay('(UTC+00:00) LONDON'  , true , i_londonSummerTime )
    marketColor := str.contains(market, 'Closes') ? #26a69a : #ef5350
    table.cell(clock, 0, 3, "█", text_size = textSize2, text_color = cLondon)
    table.cell(clock, 1, 3, array.get(a_majorCity, array.indexof(a_utcCity, '(UTC+00:00) LONDON'))    , text_color = color.blue, bgcolor = color.new(color.blue,  75), text_halign = text.align_left, text_size = textSize2)
    table.cell(clock, 2, 3, market                                                                    , text_color = marketColor , bgcolor =   color.new(marketColor, 75), text_halign = text.align_left, text_size = textSize2)

    market :=  f_digitalDisplay('(UTC-05:00) NEW YORK', true , i_newyorkSummerTime)
    marketColor := str.contains(market, 'Closes') ? #26a69a : #ef5350
    table.cell(clock, 0, 4, "█", text_size = textSize2, text_color = cNYork)
    table.cell(clock, 1, 4, array.get(a_majorCity, array.indexof(a_utcCity, '(UTC-05:00) NEW YORK'))  , text_color = color.blue, bgcolor = color.new(color.blue,  75), text_halign = text.align_left, text_size = textSize2)
    table.cell(clock, 2, 4, market                                                                    , text_color = marketColor , bgcolor =   color.new(marketColor, 75), text_halign = text.align_left, text_size = textSize2)

// ---------------------------------------------------------------------------------------------- //
// Forex Market Events -------------------------------------------------------------------------- //

var blabla       = 0
var label events = na

var a_lines      = array.new_line()
var a_labels     = array.new_label()
var a_fill       = array.new_linefill()

lookbackLength   = 1440 / tfMultiplier
priceHighest     = ta.highest(high, lookbackLength)
priceLowest      = ta.lowest (low , lookbackLength)
priceChangeRate  = (priceHighest - priceLowest) / priceHighest
priceLowest     := priceLowest  * (1 - priceChangeRate * oscVerticalOffset)
oscHighest       = 10 //ta.highest(osc, lookbackLength)
hight            = priceChangeRate / oscHight

if dayChange and tfMultiplier <= 60 and forex_n_cdf and forexEvents and timeframe.isintraday
    if array.size(a_lines) > 0
        for i = 1 to array.size(a_lines)
            line.delete(array.shift(a_lines))

    if array.size(a_labels) > 0
        for i = 1 to array.size(a_labels)
            label.delete(array.shift(a_labels))

    if array.size(a_fill) > 0
        for i = 1 to array.size(a_fill)
            linefill.delete(array.shift(a_fill))
    
    for barIndex = 0 to 23
        array.push(a_lines, line.new(bar_index +  barIndex      * 60 / tfMultiplier, priceLowest * (1 - (oscHighest - array.get(a_tradingVolume, barIndex)    ) *  hight / oscHighest), 
                                     bar_index + (barIndex + 1) * 60 / tfMultiplier, priceLowest * (1 - (oscHighest - array.get(a_tradingVolume, barIndex + 1)) *  hight / oscHighest), xloc.bar_index, extend.none,  #00000000, line.style_solid, 1)) // color.from_gradient(array.get(a_tradingVolume, barIndex), 1,  6, #ef5350, #26a69a), line.style_solid, 1)) // 

        array.push(a_lines, line.new(bar_index +  barIndex      * 60 / tfMultiplier, priceLowest * (1 - hight), 
                                     bar_index + (barIndex + 1) * 60 / tfMultiplier, priceLowest * (1 - hight), xloc.bar_index, extend.none, #00000000, line.style_solid, 1))

        array.push(a_fill, linefill.new(array.get(a_lines, 2 * barIndex), array.get(a_lines, 2 * barIndex + 1), color.new(color.from_gradient(array.get(a_tradingVolume, barIndex), 1,  6, #ef5350, #26a69a), 85) ))

        array.push(a_labels, label.new(bar_index + barIndex    * 60 / tfMultiplier,  priceLowest * (1 - (oscHighest - array.get(a_tradingVolume, barIndex)) *  hight / oscHighest), '', xloc.bar_index, yloc.price, #00000000, label.style_circle, #00000000, size.tiny, text.align_left, 'At this time of day\n - Trading volume is typically ' + array.get(a_forexEvents, barIndex) )  )
        array.push(a_labels, label.new(bar_index + barIndex    * 60 / tfMultiplier,  priceLowest * (1 - (oscHighest - array.get(a_tradingVolume, barIndex)) *  hight / oscHighest), '', xloc.bar_index, yloc.price, color.from_gradient(array.get(a_tradingVolume, barIndex), 1,  6, #ef5350, #26a69a), label.style_circle, #00000000, size.auto, text.align_left, '')  )//'At this time of day\n - Trading volume is typically ' + array.get(a_forexEvents, barIndex) )  )
    array.push(a_labels, label.new(bar_index + 24    * 60 / tfMultiplier,  priceLowest * (1 - (oscHighest - array.get(a_tradingVolume, 0)) *  hight / oscHighest), '', xloc.bar_index, yloc.price, #00000000, label.style_circle, #00000000, size.tiny, text.align_left, 'At this time of day\n - Trading volume is typically ' + array.get(a_forexEvents, 0) )  )
    array.push(a_labels, label.new(bar_index + 24    * 60 / tfMultiplier,  priceLowest * (1 - (oscHighest - array.get(a_tradingVolume, 0)) *  hight / oscHighest), '', xloc.bar_index, yloc.price, color.from_gradient(array.get(a_tradingVolume, 0), 1,  6, #ef5350, #26a69a), label.style_circle, #00000000, size.auto, text.align_left, '')  )//'At this time of day\n - Trading volume is typically ' + array.get(a_forexEvents, 0) )  )

    label.delete(events[1]) 
    blabla := 0
    events := label.new(bar_index,  priceLowest * (1 - hight), '', xloc.bar_index, yloc.price, color.new(color.from_gradient(array.get(a_tradingVolume, blabla), 1,  6, #ef5350, #26a69a), 75), label.style_label_up, color.white, size.tiny, text.align_left, 'At this time of day\n - Trading volume is typically ' + array.get(a_forexEvents, blabla) )
    
if ta.change(priceLowest) < 0 and not dayChange and tfMultiplier <= 60 and forex_n_cdf and forexEvents and timeframe.isintraday
    for barIndex = 0 to array.size(a_lines) - 1
        line.set_y1(array.get(a_lines , barIndex), line.get_y1(array.get(a_lines , barIndex)) - priceLowest[1] + priceLowest)
        line.set_y2(array.get(a_lines , barIndex), line.get_y2(array.get(a_lines , barIndex)) - priceLowest[1] + priceLowest)
    for barIndex = 0 to array.size(a_labels) - 1
        label.set_y(array.get(a_labels, barIndex), label.get_y(array.get(a_labels, barIndex)) - priceLowest[1] + priceLowest)
    label.set_y(events, label.get_y(events) - priceLowest[1] + priceLowest)

if ta.change(time('60')) and not dayChange and tfMultiplier <= 60 and forex_n_cdf and forexEvents and timeframe.isintraday
    blabla += 1
    label.set_tooltip(events, 'At this time of day\n - Trading volume is typically ' + array.get(a_forexEvents, blabla) )
    label.set_color(events, color.from_gradient(array.get(a_tradingVolume, blabla), 1,  6, #ef5350, #26a69a))
    label.set_x(events, bar_index) 
else if forex_n_cdf and tfMultiplier <= 60 and forexEvents and timeframe.isintraday
    label.set_x(events, bar_index)

// ---------------------------------------------------------------------------------------------- //
// Process Forex Markets ------------------------------------------------------------------------ //

f_getSessionInfo(_utc, _dst) =>
    TZ  = 'Etc/UTC'
    DST = _dst ? 1 : 0
    utcTime = (array.get(a_utcTimeOffset, array.indexof(a_utcCity, _utc)) + DST) * 3600000 + time
    h = math.floor(utcTime / 3600000) % 24
    A = dayofweek(int(utcTime), TZ)

    if A != 1 and A != 7
        fxO = array.get(a_forexOpenH , array.indexof(a_forexMarket, _utc))
        fxC = array.get(a_forexCloseH, array.indexof(a_forexMarket, _utc))

        if h >= fxO and h < fxC
            true
        else
            false
    else
        false

f_calculateOpeningRange(_numberOfDataRequired) =>
    var orH = 0., var orL = 0.
    [aH, aL] = request.security_lower_tf(syminfo.tickerid, '1', [high, low])
    //f_drawLabelX(bar_index, high, str.tostring(aH, format.mintick), xloc.bar_index, yloc.price, color.gray, label.style_label_down, color.white, size.normal, text.align_left, str.tostring(array.size(aH)))
    //f_drawLabelX(bar_index, low , str.tostring(aL, format.mintick), xloc.bar_index, yloc.price, color.gray, label.style_label_up  , color.white, size.normal, text.align_left, str.tostring(low , format.mintick))

    if array.size(aH) > 0
        if _numberOfDataRequired >= tfMultiplier 
            orH := array.max(aH)
            orL := array.min(aL)

        else if _numberOfDataRequired < tfMultiplier
            if array.size(aH) >= _numberOfDataRequired
                orH := array.get(aH, 0)
                orL := array.get(aL, 0)

                for c = 1 to _numberOfDataRequired - 1
                    orH := math.max(array.get(aH, c), orH)
                    orL := math.min(array.get(aL, c), orL)
            else 
                orH := array.max(aH)
                orL := array.min(aL)
    [orH, orL]

f_processForex(_show, _utc, _dst, _color, _bg) =>
    if _show != 'None' 
        var pro = 0., var prh = 0., var prl = 0.
        var sessionStartBar = 0
        var line lnh = na, var line lnl = na
        var count = 0

        session = f_getSessionInfo(_utc, _dst)
        
        if session and session != session[1] 
            sessionStartBar := bar_index
            sessionEndbar    = sessionStartBar + int( ( array.get(a_forexCloseH, array.indexof(a_utcCity, _utc)) - array.get(a_forexOpenH, array.indexof(a_utcCity, _utc)) ) * 60 ) / tfMultiplier

            if _show == 'Open'
                pro := open
                line.new(sessionStartBar, pro, sessionEndbar, pro, xloc.bar_index, extend.none, _color, line.style_solid, 2)

            if _show == 'Range'
                prh := high
                prl := low
                lnh := line.new(sessionStartBar, prh , sessionEndbar, prh , xloc.bar_index, extend.none, _color, line.style_solid, 1)
                lnl := line.new(sessionStartBar, prl , sessionEndbar, prl , xloc.bar_index, extend.none, _color, line.style_solid, 1)
                
            if _show == '5m Opening Range'
                [prh5, prl5] = f_calculateOpeningRange(5)
                lnh := line.new(sessionStartBar, prh5, sessionEndbar, prh5, xloc.bar_index, extend.none, _color, line.style_solid, 1)
                lnl := line.new(sessionStartBar, prl5, sessionEndbar, prl5, xloc.bar_index, extend.none, _color, line.style_solid, 1)

            if _show == '15m Opening Range'
                [prh5, prl5] = f_calculateOpeningRange(15)
                lnh := line.new(sessionStartBar, prh5, sessionEndbar, prh5, xloc.bar_index, extend.none, _color, line.style_solid, 1)
                lnl := line.new(sessionStartBar, prl5, sessionEndbar, prl5, xloc.bar_index, extend.none, _color, line.style_solid, 1)

            if _bg
                linefill.new(lnh, lnl, color.new(_color, 89))

            if _show == 'Channel'
                [prh60, prl60] = f_calculateOpeningRange(60)
                lnh := line.new(sessionStartBar, prh60, sessionStartBar + (_utc == '(UTC+01:00) FRANKFURT' ? 900 : 1320) / tfMultiplier, prh60, xloc.bar_index, extend.none, _color, line.style_solid, 1)
                lnl := line.new(sessionStartBar, prl60, sessionStartBar + (_utc == '(UTC+01:00) FRANKFURT' ? 900 : 1320) / tfMultiplier, prl60, xloc.bar_index, extend.none, _color, line.style_solid, 1)

                linefill.new(lnh, lnl, color.new(_color, 89))

            if (count == 5 / tfMultiplier and _show == '5m Opening Range') or (count == 15 / tfMultiplier and _show == '15m Opening Range') or (count == 60 / tfMultiplier and _show == 'Channel')
                count := 0
                
        if session
            if _show == 'Range'
                line.set_y1(lnh, math.max(high, line.get_y1(lnh))), line.set_y2(lnh, math.max(high, line.get_y2(lnh)))
                line.set_y1(lnl, math.min(low , line.get_y1(lnl))), line.set_y2(lnl, math.min(low , line.get_y2(lnl)))
            
            if (_show == '5m Opening Range' and count < 5 / tfMultiplier and tfMultiplier < 5) or (_show == '15m Opening Range' and count < 15 / tfMultiplier and tfMultiplier < 15) or (_show == 'Channel' and count < 60 / tfMultiplier and tfMultiplier < 60)
                line.set_y1(lnh, math.max(high, line.get_y1(lnh))), line.set_y2(lnh, math.max(high, line.get_y2(lnh)))
                line.set_y1(lnl, math.min(low , line.get_y1(lnl))), line.set_y2(lnl, math.min(low , line.get_y2(lnl)))
                count += 1

if timeframe.isintraday and forex_n_cdf
    f_processForex(openingChannel != 'None' ? 'Channel' : 'None', openingChannel == 'European Session' ? '(UTC+01:00) FRANKFURT' : '(UTC+09:00) TOKYO', openingChannel == 'European Session' ? i_frankfurtSummerTime : false, cChannel, true)
    f_processForex(openingChannel == 'Both' ? 'Channel' : 'None', '(UTC+01:00) FRANKFURT', i_frankfurtSummerTime, cChannel, true)

    f_processForex(shSydney   , '(UTC+10:00) SYDNEY'   , i_sydneySummerTime   , cSydney   , bgSydney   )
    f_processForex(shTokyo    , '(UTC+09:00) TOKYO'    , false                , cTokyo    , bgTokyo    )
    f_processForex(shFrankfurt, '(UTC+01:00) FRANKFURT', i_frankfurtSummerTime, cFrankfurt, bgFrankfurt)
    f_processForex(shLondon   , '(UTC+00:00) LONDON'   , i_londonSummerTime   , cLondon   , bgLondon   )
    f_processForex(shNYork    , '(UTC-05:00) NEW YORK' , i_newyorkSummerTime  , cNYork    , bgNYork    )

// ---------------------------------------------------------------------------------------------- //
// Day Open

var dayHistoryBar = 0
var dayStartBar   = 0
var line  lnhd = na, var line  lnld = na
var label lbhd = na, var label lbld = na
var countd = 0

atrRange = f_getATR(dayATRLength, 'D') 
[H1, L1] = f_calculatePreviousRange('D')

if dayChange and  (dayOpen != 'None' or dayRefLevels != 'None') and timeframe.isintraday
    dayHistoryBar := nz(dayStartBar) ? dayStartBar : bar_index
    dayStartBar   := bar_index
    dayEndBar      = 2 * dayStartBar - dayHistoryBar
    O0 = open

    if dayOpen == 'Open'
        f_drawOnlyLineX(bar_index, O0, dayEndBar, open, xloc.bar_index, extend.none, dayColor, line.style_solid, 1)
        f_drawLabelX(dayEndBar, O0, 'DO - ' + str.tostring(O0, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, dayColor, size.normal, text.align_left, '')
        //f_drawOnlyLineX(bar_index, open, dayEndBar, open, xloc.bar_index, extend.none, color.aqua, line.style_solid, 1)
        //f_drawLabelX(dayEndBar, open, 'DO - ' + str.tostring(open, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, color.aqua, size.normal, text.align_left, '')

    if dayOpen == '15m Opening Range'
        [prh, prl] = f_calculateOpeningRange(15)
        lnhd := line.new(dayStartBar, prh, dayEndBar, prh, xloc.bar_index, extend.none, dayColor, line.style_solid, 1)
        lnld := line.new(dayStartBar, prl, dayEndBar, prl, xloc.bar_index, extend.none, dayColor, line.style_solid, 1)

        linefill.new(lnhd, lnld, color.new(dayColor, 89))

    if dayOpen == '5m Opening Range'
        [prh, prl] = f_calculateOpeningRange(5)
        lnhd := line.new(dayStartBar, prh, dayEndBar, prh, xloc.bar_index, extend.none, dayColor, line.style_solid, 1)
        lnld := line.new(dayStartBar, prl, dayEndBar, prl, xloc.bar_index, extend.none, dayColor, line.style_solid, 1)

        linefill.new(lnhd, lnld, color.new(dayColor, 89))

    if dayOpen == 'First Hour Opening Range'
        [prh, prl] = f_calculateOpeningRange(60)
        lnhd := line.new(dayStartBar, prh, dayEndBar, prh, xloc.bar_index, extend.none, dayColor, line.style_solid, 1)
        lnld := line.new(dayStartBar, prl, dayEndBar, prl, xloc.bar_index, extend.none, dayColor, line.style_solid, 1)
        //label.delete(lbhd[1]), label.delete(lbld[1])
        //lbhd := label.new(dayEndBar, prh60, 'DORH - ' + str.tostring(open, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, color.aqua, size.normal, text.align_left, '')
        //lbld := label.new(dayEndBar, prl60, 'DORL - ' + str.tostring(open, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, color.aqua, size.normal, text.align_left, '')

        linefill.new(lnhd, lnld, color.new(dayColor, 89))
    
    if (countd == 60 / tfMultiplier and dayOpen == 'First Hour Opening Range') or (countd == 15 / tfMultiplier and dayOpen == '15m Opening Range') or (countd == 5 / tfMultiplier and dayOpen == '5m Opening Range')
        countd := 0

    if dayRefLevels != 'None'
        litStat = '\n\nAverage True Range = ' + str.tostring(atrRange, format.mintick) + 
                  '\nPrevious Day Range = '   + str.tostring(H1 - L1 , format.mintick) +
                  '\n   -Previous Day High = '  + str.tostring(H1, format.mintick) +
                  '\n   -Previous Day Low  = '  + str.tostring(L1, format.mintick)

        rangee = dayRefLevels == 'Average True Range (ATR)' ? atrRange : H1 - L1

        f_drawLineX (dayStartBar, O0 + rangee * dayRefLevel1, dayEndBar, O0 + rangee * dayRefLevel1, xloc.bar_index, extend.none, dayRefColor, line.style_dotted,  2)
        f_drawLabelX(dayEndBar  , O0 + rangee * dayRefLevel1, 'R1 - ' + str.tostring(O0 + rangee * dayRefLevel1, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, dayRefColor, size.normal, text.align_left, 'First Reference Level : '  + str.tostring(O0 + rangee * dayRefLevel1, format.mintick) + litStat)

        f_drawLineX (dayStartBar, O0 - rangee * dayRefLevel1, dayEndBar, O0 - rangee * dayRefLevel1, xloc.bar_index, extend.none, dayRefColor, line.style_dotted,  2)
        f_drawLabelX(dayEndBar  , O0 - rangee * dayRefLevel1, 'S1 - ' + str.tostring(O0 - rangee * dayRefLevel1, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, dayRefColor, size.normal, text.align_left, 'First Reference Level : '  + str.tostring(O0 - rangee * dayRefLevel1, format.mintick) + litStat)

        if dayRefLevel2 > dayRefLevel1
            f_drawLineX (dayStartBar, O0 + rangee * dayRefLevel2, dayEndBar, O0 + rangee * dayRefLevel2, xloc.bar_index, extend.none, dayRefColor, line.style_dotted,  2)
            f_drawLabelX(dayEndBar  , O0 + rangee * dayRefLevel2, 'R2 - ' + str.tostring(O0 + rangee * dayRefLevel2, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, dayRefColor, size.normal, text.align_left, 'Second Reference Level : ' + str.tostring(O0 + rangee * dayRefLevel2, format.mintick) + litStat)

            f_drawLineX (dayStartBar, O0 - rangee * dayRefLevel2, dayEndBar, O0 - rangee * dayRefLevel2, xloc.bar_index, extend.none, dayRefColor, line.style_dotted,  2)
            f_drawLabelX(dayEndBar  , O0 - rangee * dayRefLevel2, 'S2 - ' + str.tostring(O0 - rangee * dayRefLevel2, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, dayRefColor, size.normal, text.align_left, 'Second Reference Level : ' + str.tostring(O0 - rangee * dayRefLevel2, format.mintick) + litStat)


if not dayChange and timeframe.isintraday and (dayOpen == 'First Hour Opening Range' and countd < 60 / tfMultiplier and tfMultiplier < 60)  or (dayOpen == '15m Opening Range' and countd < 15 / tfMultiplier and tfMultiplier < 15) or (dayOpen == '5m Opening Range' and countd < 5 / tfMultiplier and tfMultiplier < 5)
    line.set_y1(lnhd, math.max(high, line.get_y1(lnhd))), line.set_y2(lnhd, math.max(high, line.get_y2(lnhd)))
    line.set_y1(lnld, math.min(low , line.get_y1(lnld))), line.set_y2(lnld, math.min(low , line.get_y2(lnld)))
    label.set_y(lbhd, math.max(high, line.get_y1(lnhd))), label.set_y(lbld, math.min(low , line.get_y2(lnld)))
    countd += 1

// ---------------------------------------------------------------------------------------------- //
// Week - Month Open

var line  lnow = na, var label lbow = na

if weekOpen
    if weekChange
        line.delete(lnow[1]), label.delete(lbow[1])
        lnow := line.new(bar_index, open, last_bar_index + 3, open, xloc.bar_index, extend.none, weekColor, line.style_solid, 1)
        lbow := label.new(last_bar_index + 3, open, 'WO - ' + str.tostring(open, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, weekColor, size.normal, text.align_left, '')
    else
        line.set_x2(lnow, last_bar_index + 3), label.set_x(lbow, last_bar_index + 3)

var line  lnom = na, var label lbom = na

if monthOpen
    if monthChange
        line.delete(lnom[1]), label.delete(lbom[1])
        lnom := line.new(bar_index, open, last_bar_index + 3, open, xloc.bar_index, extend.none, monthColor, line.style_solid, 1)
        lbom := label.new(last_bar_index + 3, open, 'MO - ' + str.tostring(open, format.mintick), xloc.bar_index, yloc.price, #00000000, label.style_label_left, monthColor, size.normal, text.align_left, '')
    else
        line.set_x2(lnom, last_bar_index + 3), label.set_x(lbom, last_bar_index + 3)

// ---------------------------------------------------------------------------------------------- //
// VWAP ----------------------------------------------------------------------------------------- //

autoAnchor = timeframe.change(vwapAnchor)
interAnchor = startDate == time
anchor = htf_tf == 'Interactive' ? interAnchor : autoAnchor
[vwap, upper, lower] = ta.vwap(vwapSource, anchor, stdevMult1)

plot(isVwap  ? vwap : na, 'VWAP Line',  anchor ? na : #0496ff)
pu1 = plot(isVwap  and vwapBand1 ? upper : na, 'VWAP Bands Upper #1', anchor ? na : #4caf50)
pl1 = plot(isVwap  and vwapBand1 ? lower : na, 'VWAP Bands Lower #1', anchor ? na : #4caf50)
fill(pu1, pl1, title="VWAP Bands Fill #1", color= isVwap  and vwapBand1 ? color.new(color.green, 95)  : na)

pu2 = plot(isVwap  and vwapBand2 ? vwap + (upper - vwap)/stdevMult1 * stdevMult2 : na, 'VWAP Bands Upper #2', anchor ? na : #808000)
pl2 = plot(isVwap  and vwapBand2 ? vwap - (vwap - lower)/stdevMult1 * stdevMult2 : na, 'VWAP Bands Lower #2', anchor ? na : #808000)
fill(pu2, pl2, title="VWAP Bands Fill #2", color= isVwap  and vwapBand2 ? color.new(color.olive, 95) : na)

//plot(isVwap  and vwapBand3 ? vwap + (upper - vwap)/stdevMult1 * stdevMult3 : na, '', anchor ? na : #00897b)
//plot(isVwap  and vwapBand3 ? vwap - (vwap - lower)/stdevMult1 * stdevMult3 : na, '', anchor ? na : #00897b)

// ---------------------------------------------------------------------------------------------- //
// Pivot Points High Low ------------------------------------------------------------------------ //

nzVolume          = nz(volume)
pvtHigh           = ta.pivothigh(pvtLength, pvtLength)
pvtLow            = ta.pivotlow (pvtLength, pvtLength)
proceed           = not na(pvtHigh) or not na(pvtLow)

pvtLengthTemp     = 3
pvtHighTemp       = ta.pivothigh(pvtLengthTemp, pvtLengthTemp)
pvtLowTemp        = ta.pivotlow (pvtLengthTemp, pvtLengthTemp)
proceedTemp       = not na(pvtHighTemp) or not na(pvtLowTemp)

var x1            = 0
var x2            = 0
var x2Temp        = 0

var pvtHigh1      = 0.
var pvtLow1       = 0.
var pvtHigh1Temp  = 0.
var pvtLow1Temp   = 0.

//var pvtLast       = ''

if proceed
    x1 := x2
    x2 := bar_index

if proceedTemp
    x2Temp := bar_index

profileLength = x2 - x1
profileLengthTemp = x2Temp - pvtLengthTemp - x2 + pvtLength

var label tempHigh = na
var label tempLow  = na

if dispPVT
    if not na(pvtHigh)
        tradedVolume = f_getTradedVolume(profileLength, proceed, pvtLength)
        f_drawOnlyLabelX(bar_index[pvtLength], pvtHigh, (pvtPrice ? str.tostring(pvtHigh, format.mintick) :  '') + (pvtChange ? (pvtPrice ? ' ↑ %' : '↑ %') + str.tostring((pvtHigh - pvtLow1) * 100 / pvtLow1 , '#.##') : '') + (pvtVolume and  nzVolume ? (pvtPrice or pvtChange ? '\n' : '') + str.tostring(tradedVolume, format.volume) : ''), xloc.bar_index, yloc.price, chart.fg_color, label.style_label_down, chart.bg_color, (not pvtPrice and not pvtChange and not pvtVolume ? size.tiny : pvtTextSize), text.align_center, 'Pivot High : ' + str.tostring(pvtHigh, format.mintick) + '\n -Price Change : ↑ %' + str.tostring((pvtHigh - pvtLow1) * 100 / pvtLow1 , '#.##') + (nzVolume ? '\n -Traded Volume : ' + str.tostring(tradedVolume, format.volume)  + ' (' + str.tostring(profileLength - 1) + ' bars)\n  *Average Volume/Bar : ' + str.tostring(tradedVolume / (profileLength - 1), format.volume) : '') + '\n\nNumber of bars : ' + str.tostring(profileLength) )
        pvtHigh1 := pvtHigh
        //pvtLast  := 'H'
        label.delete(tempHigh[1])
        if x2 - pvtLength > x2Temp - pvtLengthTemp
            label.delete(tempLow[1])

    if not na(pvtLow)
        tradedVolume = f_getTradedVolume(profileLength, proceed, pvtLength)
        f_drawOnlyLabelX(bar_index[pvtLength], pvtLow , (pvtPrice ? str.tostring(pvtLow , format.mintick) :  '') + (pvtChange ? (pvtPrice ? ' ↓ %' : '↓ %') + str.tostring((pvtHigh1 - pvtLow) * 100 / pvtHigh1, '#.##') : '') + (pvtVolume and  nzVolume ? (pvtPrice or pvtChange ? '\n' : '') + str.tostring(tradedVolume, format.volume) : ''), xloc.bar_index, yloc.price, chart.fg_color, label.style_label_up  , chart.bg_color, (not pvtPrice and not pvtChange and not pvtVolume ? size.tiny : pvtTextSize), text.align_center, 'Pivot Low : '  + str.tostring(pvtLow , format.mintick) + '\n -Price Change : ↓ %' + str.tostring((pvtHigh1 - pvtLow) * 100 / pvtHigh1, '#.##') + (nzVolume ? '\n -Traded Volume : ' + str.tostring(tradedVolume, format.volume) + ' (' + str.tostring(profileLength - 1) + ' bars)\n  *Average Volume/Bar : ' + str.tostring(tradedVolume / (profileLength - 1), format.volume) : '') + '\n\nNumber of bars : ' + str.tostring(profileLength) )
        pvtLow1  := pvtLow
        //pvtLast  := 'L'
        label.delete(tempLow[1])
        if x2 - pvtLength > x2Temp - pvtLengthTemp// ???
            label.delete(tempHigh[1])

    if not na(pvtHighTemp) //and pvtLast  == 'L' 
        if pvtHighTemp > pvtHigh1Temp// or pvtHighTemp > pvtHigh1 
            label.delete(tempHigh[1])
            tradedVolume = f_getTradedVolume(profileLengthTemp, proceedTemp, pvtLengthTemp)
            tempHigh := label.new(bar_index[pvtLengthTemp], pvtHighTemp, '* ' + (pvtPrice ? str.tostring(pvtHighTemp, format.mintick) :  '') + (pvtChange ? (pvtPrice ? ' ↑ %' : '↑ %') + str.tostring((pvtHighTemp - pvtLow1) * 100 / pvtLow1 , '#.##') : '') + (pvtVolume and  nzVolume ? (pvtPrice or pvtChange ? '\n' : '') + str.tostring(tradedVolume, format.volume) : ''), xloc.bar_index, yloc.price, #284aa9, label.style_label_down, color.white, (not pvtPrice and not pvtChange and not pvtVolume ? size.tiny : pvtTextSize), text.align_center, 'Temporary Pivot High : ' + str.tostring(pvtHighTemp, format.mintick) + '\n -Price Change : ↑ %' + str.tostring((pvtHighTemp - pvtLow1) * 100 / pvtLow1 , '#.##') + (nzVolume ? '\n -Traded Volume : ' + str.tostring(tradedVolume, format.volume)  + ' (' + str.tostring(profileLengthTemp - 1) + ' bars)\n  *Average Volume/Bar : ' + str.tostring(tradedVolume / (profileLengthTemp - 1), format.volume) : '') + '\n\nNumber of bars\n since last confirmed Pivot High/Low : ' + str.tostring(profileLengthTemp) + '\n\nWarning : subject to repaint, not a confirmed Pivot Level or Signal' )
        pvtHigh1Temp := pvtHighTemp
    
    if high > pvtHigh1Temp
        label.delete(tempHigh[1])

    if not na(pvtLowTemp) //and pvtLast  == 'H'
        if pvtLowTemp < pvtLow1Temp// or pvtLowTemp < pvtLow1
            tradedVolume = f_getTradedVolume(profileLengthTemp, proceedTemp, pvtLengthTemp)
            label.delete(tempLow[1])
            tempLow := label.new(bar_index[pvtLengthTemp], pvtLowTemp, '* ' + (pvtPrice ? str.tostring(pvtLowTemp, format.mintick) :  '') + (pvtChange ? (pvtPrice ? ' ↓ %' : '↓ %') + str.tostring((pvtHigh1 - pvtLowTemp) * 100 / pvtLowTemp , '#.##') : '') + (pvtVolume and  nzVolume ? (pvtPrice or pvtChange ? '\n' : '') + str.tostring(tradedVolume, format.volume) : ''), xloc.bar_index, yloc.price, #284aa9, label.style_label_up, color.white, (not pvtPrice and not pvtChange and not pvtVolume ? size.tiny : pvtTextSize), text.align_center, 'Temporary Pivot Low : ' + str.tostring(pvtLowTemp, format.mintick) + '\n -Price Change : ↓ %' + str.tostring((pvtHigh1 - pvtLowTemp) * 100 / pvtHigh1 , '#.##') + (nzVolume ? '\n -Traded Volume : ' + str.tostring(tradedVolume, format.volume)  + ' (' + str.tostring(profileLengthTemp - 1) + ' bars)\n  *Average Volume/Bar : ' + str.tostring(tradedVolume / (profileLengthTemp - 1), format.volume) : '') + '\n\nNumber of bars\n since last confirmed Pivot High/Low : ' + str.tostring(profileLengthTemp) + '\n\nWarning : subject to repaint, not a confirmed Pivot Level or Signal')
        pvtLow1Temp  := pvtLowTemp

    if low < pvtLow1Temp
        label.delete(tempLow[1])

// ---------------------------------------------------------------------------------------------- //
// Moving Average ------------------------------------------------------------------------------- //

plot(maDisplay ? f_getMA(maSource, maLength, maType) : na, 'Moving Average #1', maColor)
plot(maDisplay ? f_getMA(maSource2, maLength2, maType2) : na, 'Moving Average #2', maColor2, 2)


// ---------------------------------------------------------------------------------------------- //

var table logo = table.new(position.bottom_right, 1, 1)
if barstate.islast
    table.cell(logo, 0, 0, '☼☾  ', text_size=size.normal, text_color=color.teal)
