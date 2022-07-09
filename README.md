//@version=4

study(title="Heatmap Volume [xdecow]", shorttitle="HVol [xdecow]", max_bars_back=2000, format=format.volume)



//------------------------------------------------------------------------------
// Inputs


length          = input(610, title="MA Length", type=input.integer, minval=2)
slength         = input(610, title='Std Length', type=input.integer, minval=2)
cmode           = input('Heatmap', 'Color Mode', options=['Heatmap', 'Up/Down'])
zmode           = input('Backgrounds', 'Display Heatmap Zones as', options=['None', 'Lines', 'Backgrounds', 'Both'])
bcolor_enabled  = input(true, 'Colored bars')
osc             = input(false, 'Show as oscillator')

thresholdExtraHigh  = input(4, title="Extra High Volume Threshold", type=input.float)
thresholdHigh       = input(2.5, title="High Volume Threshold", type=input.float)
thresholdMedium     = input(1, title="Medium Volume Threshold", type=input.float)
thresholdNormal     = input(-0.5, title="Normal Volume Threshold", type=input.float)



//------------------------------------------------------------------------------
// Colors


// heatmap colors
chm1 = #ff0000 // extra high red
chm2 = #ff7800 // high orange
chm3 = #ffcf03 // medium yellow
chm4 = #a0d6dc // normal
chm5 = #1f9cac // low


// heatmap colors
chmthresholdExtraHigh = input(chm1, 'Heatmap Extra High')
chmthresholdHigh      = input(chm2, 'Heatmap High')
chmthresholdMedium    = input(chm3, 'Heatmap Medium')
chmthresholdNormal    = input(chm4, 'Heatmap Normal')
chmthresholdLow       = input(chm5, 'Heatmap Low')


// up colors
cupthresholdExtraHigh = input(#00FF00, 'Up Extra High')
cupthresholdHigh      = input(#30FF30, 'Up High')
cupthresholdMedium    = input(#60FF60, 'Up Medium')
cupthresholdNormal    = input(#8FFF8F, 'Up Normal')
cupthresholdLow       = input(#BFFFBF, 'Up Low')


// down colors
cdnthresholdExtraHigh = input(#FF0000, 'Down Extra High')
cdnthresholdHigh      = input(#FF3030, 'Down High')
cdnthresholdMedium    = input(#FF6060, 'Down Medium')
cdnthresholdNormal    = input(#FF8F8F, 'Down Normal')
cdnthresholdLow       = input(#FFBFBF, 'Down Low')


// threshold colors
cthresholdExtraHighUp = cmode == 'Heatmap' ? chmthresholdExtraHigh : cupthresholdExtraHigh
cthresholdHighUp      = cmode == 'Heatmap' ? chmthresholdHigh : cupthresholdHigh
cthresholdMediumUp    = cmode == 'Heatmap' ? chmthresholdMedium : cupthresholdMedium
cthresholdNormalUp    = cmode == 'Heatmap' ? chmthresholdNormal : cupthresholdNormal
cthresholdLowUp       = cmode == 'Heatmap' ? chmthresholdLow : cupthresholdLow

cthresholdExtraHighDn = cmode == 'Heatmap' ? chmthresholdExtraHigh : cdnthresholdExtraHigh
cthresholdHighDn      = cmode == 'Heatmap' ? chmthresholdHigh : cdnthresholdHigh
cthresholdMediumDn    = cmode == 'Heatmap' ? chmthresholdMedium : cdnthresholdMedium
cthresholdNormalDn    = cmode == 'Heatmap' ? chmthresholdNormal : cdnthresholdNormal
cthresholdLowDn       = cmode == 'Heatmap' ? chmthresholdLow : cdnthresholdLow



//------------------------------------------------------------------------------
// Calcs


length := length > bar_index + 1 ? bar_index + 1 : length
slength := slength > bar_index + 1 ? bar_index + 1 : slength


pstdev(Series, Period) =>
    mean = sum(Series, Period) / Period
    summation = 0.0
    for i=0 to Period-1
        sampleMinusMean = nz(Series[i]) - mean
        summation := summation + sampleMinusMean * sampleMinusMean
    return = sqrt(summation / Period)


mean    = sma(volume, length)
std     = pstdev(volume, slength)
stdbar  = (volume - mean) / std
dir     = close > open
v       = osc ? volume - mean : volume
mosc    = osc ? 0 : mean


bcolor = stdbar > thresholdExtraHigh ? dir ? cthresholdExtraHighUp : cthresholdExtraHighDn :
  stdbar > thresholdHigh  ? dir ? cthresholdHighUp : cthresholdHighDn :
  stdbar > thresholdMedium ? dir ? cthresholdMediumUp : cthresholdMediumDn :
  stdbar > thresholdNormal ? dir ? cthresholdNormalUp : cthresholdNormalDn :
  dir ? cthresholdLowUp : cthresholdLowDn


// heatmap lines
zshow_lines = zmode == 'Lines' or zmode == 'Both'
zshow_backgrounds = zmode == 'Backgrounds' or zmode == 'Both'

tst = highest(v, min(300, bar_index+1)) * 9999
ts0 = osc ? lowest(v, min(300, bar_index+1)) * 9999 : 0
ts1 = std * thresholdExtraHigh + mosc
ts2 = std * thresholdHigh + mosc
ts3 = std * thresholdMedium + mosc
ts4 = std * thresholdNormal + mosc


//------------------------------------------------------------------------------
// Plots


barcolor(bcolor_enabled ? bcolor : na, editable=false)


// hidden heatmap lines to fill
pt = plot(zshow_backgrounds ? tst : na, color=na, display=display.none, editable=false)
p0 = plot(zshow_backgrounds ? ts0 : na, color=na, display=display.none, editable=false)

p1 = plot(zshow_backgrounds ? ts1 : na, color=na, display=display.none, editable=false)
p2 = plot(zshow_backgrounds ? ts2 : na, color=na, display=display.none, editable=false)
p3 = plot(zshow_backgrounds ? ts3 : na, color=na, display=display.none, editable=false)
p4 = plot(zshow_backgrounds ? ts4 : na, color=na, display=display.none, editable=false)


// heatmap fills
tpf = 85
fill(pt, p1, chm1, transp=tpf, title='Extra High heatmap zone')
fill(p1, p2, chm2, transp=tpf, title='High heatmap zone')
fill(p2, p3, chm3, transp=tpf, title='Medium heatmap zone')
fill(p3, p4, chm4, transp=tpf, title='Normal heatmap zone')
fill(p4, p0, chm5, transp=tpf, title='Low heatmap zone')


// volume
plot(v, color=bcolor, style=plot.style_columns, title='Volume', transp=0, editable=false)


// moving average
plot(osc ? na : mean, color=#000000, linewidth=2, title='Moving Average', style=plot.style_line, transp=0, display=display.none)


// heatmap lines
tpp = 50
plot(zshow_lines ? ts1 : na, color=chm1, title='Extra High heatmap line', transp=tpp)
plot(zshow_lines ? ts2 : na, color=chm2, title='High heatmap line', transp=tpp)
plot(zshow_lines ? ts3 : na, color=chm3, title='Medium heatmap line', transp=tpp)
plot(zshow_lines ? ts4 : na, color=chm4, title='Normal heatmap line', transp=tpp)



//------------------------------------------------------------------------------
// Alerts


conditionExtraHigh  = stdbar > thresholdExtraHigh
conditionHigh       = stdbar <= thresholdExtraHigh and stdbar > thresholdHigh 
conditionMedium     = stdbar <= thresholdHigh and stdbar > thresholdMedium
conditionNormal     = stdbar <= thresholdMedium and stdbar > thresholdNormal
conditionLow        = stdbar <= thresholdNormal

alertcondition(conditionExtraHigh, title='Any Extra High Vol', message='Any Bar Extra High Volume Threshold')
alertcondition(conditionExtraHigh and open < close, title='Up Extra High', message='Up Bar Extra High Volume Threshold')
alertcondition(conditionExtraHigh and open > close, title='Down Extra High', message='Down Bar Extra High Volume Threshold')

alertcondition(conditionHigh, title='Any High Vol', message='Any Bar High Volume Threshold')
alertcondition(conditionHigh and open < close, title='Up High Vol', message='Up Bar High Volume Threshold')
alertcondition(conditionHigh and open > close, title='Down High Vol', message='Down Bar High Volume Threshold')

alertcondition(conditionMedium, title='Any Medium Vol', message='Any Bar Medium Volume Threshold')
alertcondition(conditionMedium and open < close, title='Up Medium Vol', message='Up Bar Medium Volume Threshold')
alertcondition(conditionMedium and open > close, title='Down Medium Vol', message='Down Bar Medium Volume Threshold')

