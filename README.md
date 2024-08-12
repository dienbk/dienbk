//+------------------------------------------------------------------+
//|                                                      ProjectName |
//|                                      Copyright 2020, CompanyName |
//|                                       http://www.companyname.net |
//+------------------------------------------------------------------+

#import"CalculatLots&SLPointsR5.ex5"
double CalculateLotSize(string symbol, double RiskPercent, int SLPoints);
int CalculateSLPoints(string symbol, double RiskPercent, int SLPoints);
int CalculateMaxSLPoints(string symbol, double RiskPercent, int SLPoints);
double CalculateUsedMarginForNewPosition(double lotSize, string symbol);
double CalculateMarginLevelForNewPosition(double lotSize, string symbol);
double CalculatePointValue(string symbol);

#import"4.1 CloseAllPositions R1.0.ex5"
void CloseAllPosition();
#import"6.1 BestPeroidOfTime R1.0.ex5"
string TradingSession();
#import"5.2 Getstringoftimeframe R0.ex5"
string getstringoftimeframe(ENUM_TIMEFRAMES Timeframe=PERIOD_M1);
#import"6.4 DownCountOnCurrentTimeframe R1.ex5"
string UpdateTimeLeft();
#import
input group "1.Setting For Trading"
enum Tradingmode
  {
   MANUAL=1
   ,AUTO=0
  };
Tradingmode TradingmodeSelection=MANUAL;//Trading Mode
int TrendMovingAverageArray=9;//Moving Average
ENUM_APPLIED_PRICE Type_Price=PRICE_MEDIAN;
ENUM_TIMEFRAMES TradingTimeframe=PERIOD_D1;//Trading Timeframe
string TradeSymbol="";
string AllWatch_Symbol="AUDCHF|AUDJPY|AUDNZD|AUDUSD|CADJPY|CADCHF|EURAUD|EURCAD|EURCHF|EURGBP|EURJPY|EURNZD|EURUSD|GBPAUD|GBPCAD|GBPJPY|GBPNZD|GBPCHF|GBPUSD|NZDCAD|NZDJPY|NZDUSD|NZDCHF|USDCAD|USDCHF|USDJPY|XAUUSD|ETHUSD|BTCUSD|XRPUSD|US30|US100|US500|JP225|HK50|NVIDA|TESLA|APPLE|AMAZON|MICROSOFT|META|AMD|GOOGLE|INTEL|NETFLIX|GOLD|TSM|MCDONALDS|NIKE";
string MainWatch_Symbol="AUDUSD|EURUSD|GBPUSD|NZDUSD|USDCAD|USDCHF|USDJPY|XAUUSD";
string PopularWatch_Symbol="AUDUSD|EURAUD|EURCAD|EURCHF|EURGBP|EURJPY|EURNZD|EURUSD|GBPAUD|GBPCAD|GBPJPY|GBPCHF|GBPUSD|NZDUSD|USDCAD|USDCHF|USDJPY|XAUUSD";
string CrytoWatch_Symbol="ETHUSD|BTCUSD|XRPUSD";
string IndexWatch_Symbol="US30|US100|US500|JP225|HK50";
string StockWatch_Symbol="NVIDA|TESLA|APPLE|AMAZON|MICROSOFT|META|AMD|GOOGLE|INTEL|NETFLIX|GOLD|TSM|MCDONALDS|NIKE";
enum SymbolforWatchlist
  {

   All=0
   ,Main=1
   ,Popular=2
   ,Cryto=3
   ,Index=4
   ,Stock=5

  };

input SymbolforWatchlist SelectedSymbolforwatchlist=Main;//Select Watchlist
input group "2.Setting For Account Management"
static double InitialBalance=200;
static datetime SetTimeCount;
input int MarginLevelLimit=800;
input group "3.Setting For Manual Panel"
input int x=0;
input int y=-50;
#include <Trade\Trade.mqh>
CTrade trade;
#resource "\\Files\\DailyTarget.wav";
#resource "\\Files\\Margin Alert.wav";
#resource "\\Files\\Cap tien khong co watchlist.wav";
#resource "\\Files\\buystop.wav";
#resource "\\Files\\sellstop.wav";
string FastTrailingStopSelection="OFF";
string Tradingpositontype="";
int MaxPositions;
string TrailingStopSelection="OFF";
string SelectedTimeframe=getstringoftimeframe(TradingTimeframe);
int Initial_SL=600;

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
int OnInit()
  {

   string Watch_Symbol;

   switch(SelectedSymbolforwatchlist)
     {
      case 0:
         Watch_Symbol = AllWatch_Symbol;
         break;
      case 1:
         Watch_Symbol = MainWatch_Symbol;
         break;
      case 2:
         Watch_Symbol = PopularWatch_Symbol;
         break;
      case 3:
         Watch_Symbol = CrytoWatch_Symbol;
         break;
      case 4:
         Watch_Symbol = IndexWatch_Symbol;
         break;
      case 5:
         Watch_Symbol = StockWatch_Symbol;
         break;
      default:
         // Handle any other cases if needed
         break;
     }


   string Watchlistarray[],DeleteAllsymbolbutton[];
   int NoOfsymbolsintoWatchlist= StringSplit(Watch_Symbol, '|', Watchlistarray);
   int NoofdeleteSymbol=StringSplit(AllWatch_Symbol,'|', DeleteAllsymbolbutton);

// Deselect all symbols except the current one

   for(int i=0; i<= SymbolsTotal(true);i++)
     {
      if(SymbolName(i,true)!=_Symbol)
         SymbolSelect(SymbolName(i,true),false);
     }

// Delete existing buttons
   for(int i=0; i< NoofdeleteSymbol;i++)
     {
      ObjectDelete(ChartID(),DeleteAllsymbolbutton[i]);
     }

// Create buttons for each symbol in the watchlist

   for(int i=0; i< NoOfsymbolsintoWatchlist;i++)
     {

      SymbolSelect(Watchlistarray[i],true);
      if(i<11)
         CreateButton(ChartID(),0,Watchlistarray[i],OBJ_BUTTON,clrAliceBlue,Watchlistarray[i],0,2+i*61,60,501,25);
      else
         if(i>=11 && i<22)
            CreateButton(ChartID(),0,Watchlistarray[i],OBJ_BUTTON,clrAliceBlue,Watchlistarray[i],0,2+(i-11)*61,60,526,25);
         else
            if(i>=22& i<33)
               CreateButton(ChartID(),0,Watchlistarray[i],OBJ_BUTTON,clrAliceBlue,Watchlistarray[i],0,2+(i-22)*61,60,551,25);
            else
               if(i>=33)
                  CreateButton(ChartID(),0,Watchlistarray[i],OBJ_BUTTON,clrAliceBlue,Watchlistarray[i],0,2+(i-33)*61,60,576,25);
      ObjectSetInteger(ChartID(),Watchlistarray[i],OBJPROP_STATE,false);
      ObjectSetInteger(ChartID(),Watchlistarray[i],OBJPROP_BGCOLOR,clrWhiteSmoke);

     }

   ChartSetInteger(ChartID(),CHART_COLOR_BACKGROUND,clrBlack);
   ChartSetInteger(ChartID(),CHART_SHOW_ONE_CLICK,0);
   ChartSetInteger(ChartID(),CHART_SHOW_BID_LINE,1);
   ChartSetInteger(ChartID(),CHART_COLOR_BID,clrPink);
   ChartSetInteger(ChartID(),CHART_AUTOSCROLL,1);
   ChartSetInteger(ChartID(),CHART_SHIFT,1);
// ChartSetInteger(ChartID(),CHART_SCALEFIX,1);
// ChartSetInteger(ChartID(),CHART_SCALE,1);
   ChartSetInteger(ChartID(),CHART_SHOW_VOLUMES,0);
   ChartSetInteger(ChartID(),CHART_SHOW_GRID,0);
   ChartSetInteger(ChartID(),CHART_SHOW_TRADE_HISTORY,0);

//ChartSetInteger(ChartID(),CHART_BRING_TO_TOP,1);

   if(ObjectFind(ChartID(),"RectangleLabel1")<0)
      CreateRectangleLabel(ChartID(),"RectangleLabel1",1860-x,300,100-y,350,clrWhiteSmoke);
   if(ObjectFind(ChartID(),"RectangleLabel2")<0)
      CreateRectangleLabel(ChartID(),"RectangleLabel2",1560-x,240,-y,100,clrWhiteSmoke);
   if(ObjectFind(ChartID(),"Reset_initialBalance")<0)
      CreateButton(ChartID(),0,"Reset_initialBalance",OBJ_BUTTON,clrRed,"Refresh",3,240+1370-x,50,110-y,20);
   if(ObjectFind(ChartID(),"RectangleLabel3")<0)
      CreateRectangleLabel(ChartID(),"RectangleLabel3",1860-x,670,-50-y,50,clrWheat);
   if(ObjectFind(ChartID(),"SellButton")<0)
      CreateButton(ChartID(),0,"SellButton",OBJ_BUTTON,clrRed,"SELL",3,1860-x,90,-y,50);
   if(ObjectFind(ChartID(),"TradeLotsizeEdit")<0)
      CreateEdit(0,0,"TradeLotsizeEdit",3,1770-x,60,-y,50,"0.01");
   if(ObjectFind(ChartID(),"CalculateLotsButton")<0)
      CreateButton(ChartID(),0,"CalculateLotsButton",OBJ_BUTTON,clrAqua,"Cal_Size",3,1710-x,60,-y,25);
   if(ObjectFind(ChartID(),"CalculateSLButton")<0)
      CreateButton(ChartID(),0,"CalculateSLButton",OBJ_BUTTON,clrAliceBlue,"SL:0.00",3,1710-x,60,25-y,25);
   if(ObjectFind(ChartID(),"BuyButton")<0)
      CreateButton(ChartID(),0,"BuyButton",OBJ_BUTTON,clrGreen,"BUY",3,1650-x,90,-y,50);
   if(ObjectFind(ChartID(),"BuyStop")<0)
      CreateButton(ChartID(),0,"BuyStop",OBJ_BUTTON,clrGreen,"BuyStop",3,1860-x,100,50-y,50);
   if(ObjectFind(ChartID(),"SellStop")<0)
      CreateButton(ChartID(),0,"SellStop",OBJ_BUTTON,clrRed,"SellStop",3,1760-x,100,50-y,50);
   if(ObjectFind(ChartID(),"CancelOrderStop")<0)
      CreateButton(ChartID(),0,"CancelOrderStop",OBJ_BUTTON,clrYellow,"CancelOrder",3,1660-x,100,50-y,50);
   if(ObjectFind(ChartID(),"IndicatorsRec")<0)
      CreateRectangleLabel(ChartID(),"IndicatorsRec",1320-x,131,345-y,70,clrBrown);
   if(ObjectFind(ChartID(),"MovingAverage")<0)
     {
      CreateButton(ChartID(),0,"MovingAverage",OBJ_BUTTON,clrWhiteSmoke,"Candle",3,1319-x,63,350-y,30);
      ObjectSetInteger(ChartID(),"MovingAverage",OBJPROP_STATE,true);
      ObjectSetInteger(ChartID(),"MovingAverage",OBJPROP_BGCOLOR,clrLightGreen);
      ChartIndicatorAdd(ChartID(),0,iCustom(_Symbol,PERIOD_CURRENT,"Dien\\Dien_Price+MA R3"));
     }

   if(ObjectFind(ChartID(),"RSI_Moment")<0)
     {
      CreateButton(ChartID(),0,"RSI_Moment",OBJ_BUTTON,clrWhiteSmoke,"RSI",3,1319-x,63,383-y,30);
      ObjectSetInteger(ChartID(),"RSI_Moment",OBJPROP_STATE,true);
      ObjectSetInteger(ChartID(),"RSI_Moment",OBJPROP_BGCOLOR,clrLightGreen);
     }

   if(ObjectFind(ChartID(),"ZigZag")<0)
     {
      CreateButton(ChartID(),0,"ZigZag",OBJ_BUTTON,clrWhiteSmoke,"ZigZag",3,1255-x,63,350-y,30);
      ObjectSetInteger(ChartID(),"ZigZag",OBJPROP_STATE,false);
      ObjectSetInteger(ChartID(),"ZigZag",OBJPROP_BGCOLOR,clrWhite);
     }
   if(ObjectFind(ChartID(),"ShowTradeLevels")<0)
     {
      CreateButton(ChartID(),0,"ShowTradeLevels",OBJ_BUTTON,clrWhiteSmoke,"Levels",3,1320-x,64,415-y,35);
      ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_STATE,true);
      ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_BGCOLOR,clrLightGreen);
     }

   if(ObjectFind(ChartID(),"Object_study")<0)
      CreateButton(ChartID(),0,"Object_study",OBJ_BUTTON,clrRosyBrown,"Object",3,1320-x-65,65,415-y,35);
   if(ObjectFind(ChartID(),"TRIX")<0)
     {
      CreateButton(ChartID(),0,"TRIX",OBJ_BUTTON,clrRosyBrown,"TRIX",3,1255-x,63,383-y,30);
      ObjectSetInteger(ChartID(),"TRIX",OBJPROP_STATE,false);
      ObjectSetInteger(ChartID(),"TRIX",OBJPROP_BGCOLOR,clrWhiteSmoke);
     }
   if(ObjectFind(ChartID(),"TakeProfit")<0)
      CreateButton(ChartID(),0,"TakeProfit",OBJ_BUTTON,clrGreen,"TakeProfit",3,1320-x,65,-y,50);
   if(ObjectFind(ChartID(),"TakehalfofProfit")<0)
      CreateButton(ChartID(),0,"TakehalfofProfit",OBJ_BUTTON,clrTurquoise,"1/2Profit",3,1255-x,65,-y,50);


   if(ObjectFind(ChartID(),"StopLoss")<0)
      CreateButton(ChartID(),0,"StopLoss",OBJ_BUTTON,clrRed,"StopLoss",3,1320-x,65,50-y,50);

   if(ObjectFind(ChartID(),"Stop1/3")<0)
      CreateButton(ChartID(),0,"Stop1/3",OBJ_BUTTON,clrRosyBrown,"Stop1/3",3,1255-x,65,50-y,50);

   if(ObjectFind(ChartID(),"ZeroLoss")<0)
      CreateButton(ChartID(),0,"ZeroLoss",OBJ_BUTTON,clrGold,"ZeroLoss",3,1320-x,130,100-y,50);


   if(ObjectFind(ChartID(),"CompleteTarget")<0)
      CreateButton(ChartID(),0,"CompleteTarget",OBJ_BUTTON,clrTurquoise,"CompleteTarget",3,1320-x,130,150-y,40);

   if(ObjectFind(ChartID(),"TrailingStopRectag")<0)
      CreateRectangleLabel(ChartID(),"TrailingStopRectag",1320-x,131,190-y,75,clrViolet);
   if(ObjectFind(ChartID(),"Activate_Point")<0)
      CreateEdit(0,0,"Activate_Point",3,1315-x,50,192-y,20,"0");
   if(ObjectFind(ChartID(),"ActivatePointText")<0)
      CreateLabel(ChartID(),0,"ActivatePointText",3,1260-x,192-y,"ActiPoint",9,clrBlack);
   if(ObjectFind(ChartID(),"TSL")<0)
      CreateEdit(0,0,"TSL",3,1315-x,50,215-y,20,"50");
   if(ObjectFind(ChartID(),"TSLText")<0)
      CreateLabel(ChartID(),0,"TSLText",3,1260-x,217-y,"TSPoint",9,clrBlack);
   if(ObjectFind(ChartID(),"NormalTrailingStop")<0)
     {
      CreateButton(ChartID(),0,"NormalTrailingStop",OBJ_BUTTON,clrWhiteSmoke,"Normal",3,1315-x,50,237-y,25);
      ObjectSetInteger(ChartID(),"NormalTrailingStop",OBJPROP_STATE,false);
     }
   if(ObjectFind(ChartID(),"FastTrailingStop")<0)
     {
      CreateButton(ChartID(),0,"FastTrailingStop",OBJ_BUTTON,clrWhiteSmoke,"Fast",3,1255-x+10,38,237-y,25);
      ObjectSetInteger(ChartID(),"FastTrailingStop",OBJPROP_STATE,false);
     }



   if(ObjectFind(ChartID(),"DynamicTraingStop")<0)
     {
      CreateButton(ChartID(),0,"DynamicTraingStop",OBJ_BUTTON,clrWhiteSmoke,"Dyna",3,1255-x-28,38,237-y,25);
      ObjectSetInteger(ChartID(),"DynamicTraingStop",OBJPROP_STATE,false);
     }

   if(ObjectFind(ChartID(),"PositionTypeRecg")<0)
      CreateRectangleLabel(ChartID(),"PositionTypeRecg",1320-x,131,265-y,80,clrWhiteSmoke);
   if(ObjectFind(ChartID(),"PositionTypeLabel")<0)
      CreateLabel(ChartID(),0,"PositionTypeLabel",3,1300-x,270-y,"Position-Type",10,clrGreen);

   if(ObjectFind(ChartID(),"LongOnly")<0)
     {
      CreateButton(ChartID(),0,"LongOnly",OBJ_BUTTON,clrWhiteSmoke,"LongOnly",3,1315-x,65,315-y-28,25);
      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);
      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
     }


   if(ObjectFind(ChartID(),"BuyStopSelect")<0)
     {
      CreateButton(ChartID(),0,"BuyStopSelect",OBJ_BUTTON,clrWhiteSmoke,"BuyStop",3,1315-x-65,62,315-y-28,25);
      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);
      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
     }


   if(ObjectFind(ChartID(),"SellStopSelect")<0)
     {
      CreateButton(ChartID(),0,"SellStopSelect",OBJ_BUTTON,clrWhiteSmoke,"SellStop",3,1315-x-65,62,315-y,25);
      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
     }




   if(ObjectFind(ChartID(),"ShortOnly")<0)
     {
      CreateButton(ChartID(),0,"ShortOnly",OBJ_BUTTON,clrWhiteSmoke,"ShortOnly",3,1315-x,65,315-y,25);
      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);
      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
     }



   if(ObjectFind(ChartID(),"RiskManagement_Panel_Label")<0)
      CreateRectangleLabel(ChartID(),"RiskManagement_Panel_Label",1560-x,240,100-y,350,clrWhiteSmoke);

   if(ObjectFind(ChartID(),"Account_Risk")<0)
      CreateLabel(0,0,"Account_Risk",0,310,160,"% Risk",10,clrBlack);
   if(ObjectFind(ChartID(),"P/Llabel")<0)
      CreateLabel(0,0,"P/Llabel",0,310,180,"P/L",10,clrBlack);
   if(ObjectFind(ChartID(),"SLlabel")<0)
      CreateLabel(0,0,"SLlabel",0,310,200,"StopLoss",10,clrBlack);


   if(ObjectFind(ChartID(),"%Risk")<0)
      CreateEdit(0,0,"%Risk",0,380,50,160,20,"10");

   if(ObjectFind(ChartID(),"P/LValue")<0)
      CreateEdit(0,0,"P/LValue",0,380,50,180,20,"1.5");




   if(ObjectFind(ChartID(),"SLValue")<0)
      CreateEdit(0,0,"SLValue",0,380,50,200,20,"500");





   if(ObjectFind(ChartID(),"MaxSLValue")<0)
      CreateButton(0,0,"MaxSLValue",OBJ_BUTTON,clrAqua,"MAXSL",0,480,50,200,20);


   if(ObjectFind(ChartID(),"---")<0)
      CreateLabel(0,0,"---",0,310,220,"-------------------------------------",14,clrBlack);



   if(ObjectFind(ChartID(),"SpreadMaxLabel")<0)
      CreateLabel(0,0,"SpreadMaxLabel",0,310,250,"SpreadMax",10,clrBlack);
   if(ObjectFind(ChartID(),"SpreadMax")<0)
      CreateEdit(0,0,"SpreadMax",0,380,50,250,20,"40");


   if(ObjectFind(ChartID(),"LotMaxLabel")<0)
      CreateLabel(0,0,"LotMaxLabel",0,310,270,"LotMax",10,clrBlack);
   if(ObjectFind(ChartID(),"LotMax")<0)
      CreateEdit(0,0,"LotMax",0,380,50,270,20,"0.1");


   if(ObjectFind(ChartID(),"SLpointminLabel")<0)
      CreateLabel(0,0,"SLpointminLabel",0,310,290,"SLMin",10,clrBlack);
   if(ObjectFind(ChartID(),"SLMin")<0)
      CreateEdit(0,0,"SLMin",0,380,50,290,20,"500");


   if(ObjectFind(ChartID(),"Max_postionLablel")<0)
      CreateLabel(0,0,"Max_postionLablel",0,310,310,"Max_Posi",10,clrBlack);
   if(ObjectFind(ChartID(),"Max_postion")<0)
      CreateEdit(0,0,"Max_postion",0,380,50,310,20,"2");

   if(ObjectFind(ChartID(),"----")<0)
      CreateLabel(0,0,"----",0,310,330,"-------------------------------------",14,clrBlack);
   if(ObjectFind(ChartID(),"HighestPrice")<0)
      CreateLabel(0,0,"HighestPrice",0,310,350,"HighestPrice",10,clrBlack);
   if(ObjectFind(ChartID(),"LowestPrice")<0)
      CreateLabel(0,0,"LowestPrice",0,310,370,"LowestPrice",10,clrBlack);
   if(ObjectFind(ChartID(),"OpenPrice")<0)
      CreateLabel(0,0,"OpenPrice",0,310,390,"OpenPrice",10,clrBlack);


   if(ObjectFind(ChartID(),"-----")<0)
      CreateLabel(0,0,"-----",0,310,400,"-------------------------------------",14,clrBlack);
   /*
   int AccountBalanceConfirm=MessageBox("Do you want update Account Balance","Update AccountBalance",MB_OKCANCEL);
   if(AccountBalanceConfirm==1)
   InitialBalance=NormalizeDouble(AccountInfoDouble(ACCOUNT_BALANCE),2);

   */

   if(ObjectFind(ChartID(),"TradingTimeframeButton")<0)
     {
      CreateButton(ChartID(),0,"TradingTimeframeButton",OBJ_BUTTON,clrWhiteSmoke,"D1",3,1770-x,50,230-y,20);
      ChartSetSymbolPeriod(ChartID(),_Symbol,PERIOD_D1);
      ObjectSetInteger(ChartID(),"TradingTimeframeButton",OBJPROP_STATE,false);
      ObjectSetInteger(ChartID(),"TradingTimeframeButton",OBJPROP_BGCOLOR,clrLightGreen);

     }


   ChartRedraw();
   return(INIT_SUCCEEDED);

  }




//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void OnChartEvent(const int id,
                  const long&  lparam,
                  const double& dparam,
                  const string& sparam
                 )

  {


   int SL_Space = (_Period >= PERIOD_M5) ? 300 : 200;
   double Ask = NormalizeDouble(SymbolInfoDouble(_Symbol, SYMBOL_ASK), _Digits);
   double Bid = NormalizeDouble(SymbolInfoDouble(_Symbol, SYMBOL_BID), _Digits);
   double CurrentMarginLevel = AccountInfoDouble(ACCOUNT_MARGIN_LEVEL);
   double RiskAccount = StringToDouble(ObjectGetString(0, "%Risk", OBJPROP_TEXT));
   MaxPositions = StringToInteger(ObjectGetString(ChartID(), "Max_postion", OBJPROP_TEXT));

// To save account
   double checkAccount = MathMin(AccountInfoDouble(ACCOUNT_EQUITY), AccountInfoDouble(ACCOUNT_MARGIN_FREE));
   double checkLotMax = StringToDouble(ObjectGetString(ChartID(), "LotMax", OBJPROP_TEXT));
   double checkMarginLevel = AccountInfoDouble(ACCOUNT_MARGIN_LEVEL);

   if(StringToDouble(ObjectGetString(ChartID(), "SLValue", OBJPROP_TEXT)) < StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT)))
     {
      MessageBox("Trading SL point is less than the minimum SL point. Please increase the trading SL point.", "Trading SL point Message", MB_OKCANCEL);
      ObjectSetString(ChartID(), "SLValue", OBJPROP_TEXT, IntegerToString(StringToInteger(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT)) + 100));
     }

   if(checkAccount < 200 && checkLotMax > 0.01)
      ObjectSetString(ChartID(), "LotMax", OBJPROP_TEXT, "0.01");
   else
      if(checkAccount >= 200 && checkAccount < 350 && checkLotMax > 0.03)
         ObjectSetString(ChartID(), "LotMax", OBJPROP_TEXT, "0.02");
      else
         if(checkAccount >= 350 && checkAccount < 500 && checkLotMax > 0.04)
            ObjectSetString(ChartID(), "LotMax", OBJPROP_TEXT, "0.03");

   if(checkMarginLevel < 300 && PositionsTotal() > 0)
      ObjectSetString(ChartID(), "LotMax", OBJPROP_TEXT, "0.00");

   if(checkAccount >= 500)
     {
      if(StringToDouble(ObjectGetString(0, "%Risk", OBJPROP_TEXT)) > 15)
        {
         ObjectSetString(ChartID(), "%Risk", OBJPROP_TEXT, "15");
         MessageBox("%Risk should not exceed 15%. Alert: %Risk over allowed %Risk.", "Risk Alert", MB_OK);
        }
     }
   else
     {
      if(StringToDouble(ObjectGetString(0, "%Risk", OBJPROP_TEXT)) > 20)
        {
         ObjectSetString(ChartID(), "%Risk", OBJPROP_TEXT, "20");
         MessageBox("%Risk should not exceed 20%. Alert: %Risk over allowed %Risk.", "Risk Alert", MB_OK);
        }
     }

   if(MaxPositions > 3)
     {
      MessageBox("Number of positions cannot exceed 3. Allowed maximum positions message.", "Maximum Positions Alert", 0);
      ObjectSetString(ChartID(), "Max_postion", OBJPROP_TEXT, "3");
     }



//////////////////////Calculate Lot size\\\\\\\\\\\\\\\\\\\\\\\\

   int SLPoint=StringToInteger(ObjectGetString(0,"SLValue",OBJPROP_TEXT));
   double RatioTPSL=StringToDouble(ObjectGetString(0,"P/LValue",OBJPROP_TEXT));
   double Lot_sizeTotrade=StringToDouble(ObjectGetString(ChartID(),"TradeLotsizeEdit",OBJPROP_TEXT));
   double AllowedLotsize=StringToDouble(ObjectGetString(ChartID(),"LotMax",OBJPROP_TEXT));
   double CalculatedLot_Size=StringToDouble(ObjectGetString(ChartID(),"CalculateLotsButton",OBJPROP_TEXT));



//////////////////////Filter Lot size\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\


   Lot_sizeTotrade=FilterLotSize(Lot_sizeTotrade,AllowedLotsize,CalculatedLot_Size);

   ObjectSetString(ChartID(),"TradeLotsizeEdit",OBJPROP_TEXT,(string)Lot_sizeTotrade);

   string TradingTimeFrameComment="";
//TradingTimeFrameComment=getstringoftimeframe(_Period)+" - MAN"+" SLMax-"+IntegerToString(CalculateMaxSLPoints(_Symbol,RiskAccount,SLPoint));
   TradingTimeFrameComment=IntegerToString(CalculateMaxSLPoints(_Symbol,RiskAccount,SLPoint));

   if(id==CHARTEVENT_OBJECT_CLICK)
     {

      if(sparam=="BuyButton")
         if(!ObjectGetInteger(ChartID(),"LongOnly",OBJPROP_STATE))
            MessageBox("Please click LongOnly button","Select Long/Short Message");
      if(sparam=="BuyStop")
         if(!ObjectGetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE))
            MessageBox("Please click BuyStop button","Select Buy_Stop Message");
      if(sparam=="SellStop")
         if(!ObjectGetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE))
            MessageBox("Please click SellStop button","Select Sell_Stop Message");
      if(sparam=="SellButton")
         if(!ObjectGetInteger(ChartID(),"ShortOnly",OBJPROP_STATE))
            MessageBox("Please click ShortOnly button","Select Long/Short Message");
      if(AccountInfoInteger(ACCOUNT_TRADE_MODE)==ACCOUNT_TRADE_MODE_REAL)
         if(TradingSession()=="SYD+TOKYO"||TradingSession()=="LONDON")
            if(sparam=="BuyButton"||sparam=="SellButton"||sparam=="BuyStop"||sparam=="SellStop")
               if(Lot_sizeTotrade>0.01)
                  MessageBox("You are trading on real accout, dueto trading session ( SYD+TOKYO) maxium volume can be traded 0.01","Trading session message",0);


      if(sparam=="CalculateSLButton")
         ObjectSetInteger(ChartID(),"CalculateSLButton",OBJPROP_STATE,false);
      else
         if(sparam=="CalculateLotsButton")
           {
            ObjectSetString(ChartID(),"TradeLotsizeEdit",OBJPROP_TEXT,(string)CalculateLotSize(_Symbol,RiskAccount,SLPoint));
            ObjectSetInteger(ChartID(),"CalculateLotsButton",OBJPROP_STATE,false);
           }
         else
            if(sparam=="LongOnly")
              {

               bool LongState=false;
               LongState=ObjectGetInteger(ChartID(),"LongOnly",OBJPROP_STATE);


               if(Lot_sizeTotrade<=0)
                 {
                  MessageBox("LotSize=0,Can't selecte trade button, please change SL point or %risk ","Alert Lotsize Zero");
                  ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);
                 }

               if(Lot_sizeTotrade>0)
                  if(LongState)
                    {
                     ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrLightGreen);
                     ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                     ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);
                     ObjectSetInteger(ChartID(),"Long&Short",OBJPROP_BGCOLOR,clrWhiteSmoke);
                     ObjectSetInteger(ChartID(),"Long&Short",OBJPROP_STATE,false);
                     ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                     ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);
                     ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                     ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                     ObjectDelete(0,"SLHLine3");
                     ObjectDelete(0,"SLHLine4");

                     ObjectDelete(0,"SLHLine2");
                     ObjectDelete(0,"TPHLine2");

                     HLineCreate(0,"SLHLine1",SLPoint+50,"buy");
                     HLineCreate(0,"TPHLine1",-SLPoint*RatioTPSL,"buy");
                     ObjectSetInteger(ChartID(),"TPHLine1",OBJPROP_COLOR,clrLightGreen);
                     ObjectSetInteger(ChartID(),"TPHLine1",OBJPROP_WIDTH,2);
                     ObjectSetInteger(ChartID(),"SLHLine1",OBJPROP_COLOR,clrRed);
                     ObjectSetInteger(ChartID(),"SLHLine1",OBJPROP_WIDTH,2);
                     ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_STATE,false);
                     ChartSetInteger(ChartID(),CHART_SHOW_TRADE_LEVELS,false);
                     ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_BGCOLOR,clrWhiteSmoke);
                    }
                  else
                    {

                     ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                     ObjectDelete(0,"SLHLine1");
                     ObjectDelete(0,"TPHLine1");

                    }

              }


            else
               if(sparam=="BuyStopSelect")
                 {

                  bool BuyStopState=false;
                  BuyStopState=ObjectGetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE);
                  if(Lot_sizeTotrade<=0)
                    {
                     MessageBox("LotSize=0,Can't select trade button, please change SL point or %risk ","Alert Lotsize Zero");
                     ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);
                    }
                  if(Lot_sizeTotrade>0)
                     if(BuyStopState)
                       {
                        ChartSetInteger(ChartID(),CHART_SHOW_BID_LINE,0);
                        ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrLightGreen);
                        ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                        ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);
                        ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                        ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                        ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                        ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                        ObjectSetInteger(ChartID(),"Long&Short",OBJPROP_BGCOLOR,clrWhiteSmoke);
                        ObjectSetInteger(ChartID(),"Long&Short",OBJPROP_STATE,false);

                        ObjectDelete(0,"SLHLine2");
                        ObjectDelete(0,"TPHLine2");
                        ObjectDelete(0,"SLHLine4");
                        HLineCreate(0,"SLHLine1",SLPoint,"buy");
                        HLineCreate(0,"TPHLine1",-(SLPoint+SL_Space)*RatioTPSL,"buy");

                        ObjectSetInteger(ChartID(),"TPHLine1",OBJPROP_COLOR,clrLightGreen);
                        ObjectSetInteger(ChartID(),"TPHLine1",OBJPROP_WIDTH,2);

                        HLineCreate(0,"SLHLine3",-SL_Space,"buy");
                        ObjectSetInteger(ChartID(),"SLHLine3",OBJPROP_COLOR,clrWhite);

                        ObjectSetInteger(ChartID(),"SLHLine1",OBJPROP_COLOR,clrRed);
                        ObjectSetInteger(ChartID(),"SLHLine1",OBJPROP_WIDTH,2);

                        ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_STATE,false);
                        ChartSetInteger(ChartID(),CHART_SHOW_TRADE_LEVELS,false);
                        ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_BGCOLOR,clrWhiteSmoke);

                       }
                     else
                       {

                        ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                        ObjectDelete(0,"SLHLine1");
                        ObjectDelete(0,"TPHLine1");
                        ObjectDelete(0,"SLHLine3");

                       }

                 }


               /////////////////////////////////////////////////////////////////////////////////////////////////////

               else
                  if(sparam=="ShortOnly")
                    {


                     bool ShortState=false;
                     ShortState=ObjectGetInteger(ChartID(),"ShortOnly",OBJPROP_STATE);
                     if(Lot_sizeTotrade<=0)
                       {
                        MessageBox("LotSize=0,Can't select trade button, please change SL point or %risk ","Alert Lotsize Zero");
                        ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);
                       }
                     if(Lot_sizeTotrade>0)
                        if(ShortState)
                          {
                           ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrLightGreen);
                           ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                           ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);
                           ObjectSetInteger(ChartID(),"Long&Short",OBJPROP_BGCOLOR,clrWhiteSmoke);
                           ObjectSetInteger(ChartID(),"Long&Short",OBJPROP_STATE,false);
                           ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_STATE,false);
                           ChartSetInteger(ChartID(),CHART_SHOW_TRADE_LEVELS,false);
                           ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_BGCOLOR,clrWhiteSmoke);

                           ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                           ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                           ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                           ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                           ObjectDelete(0,"SLHLine3");
                           ObjectDelete(0,"SLHLine4");

                           ObjectDelete(0,"SLHLine1");
                           ObjectDelete(0,"TPHLine1");
                           HLineCreate(0,"SLHLine2",SLPoint+50,"sell");
                           HLineCreate(0,"TPHLine2",-SLPoint*RatioTPSL,"sell");
                           ObjectSetInteger(ChartID(),"TPHLine2",OBJPROP_COLOR,clrLightGreen);
                           ObjectSetInteger(ChartID(),"TPHLine2",OBJPROP_WIDTH,2);

                          }
                        else
                          {

                           ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                           ObjectDelete(0,"SLHLine2");
                           ObjectDelete(0,"TPHLine2");

                          }
                    }


                  //////////////////////////////////////////////////////////////////

                  else
                     if(sparam=="SellStopSelect")
                       {


                        bool SellStopState=false;
                        SellStopState=ObjectGetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE);
                        if(Lot_sizeTotrade<=0)
                          {
                           MessageBox("LotSize=0,Can't select trade button, please change SL point or %risk ","Alert Lotsize Zero");
                           ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                          }
                        if(Lot_sizeTotrade>0)
                           if(SellStopState)
                             {
                              ChartSetInteger(ChartID(),CHART_SHOW_BID_LINE,0);
                              ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrLightGreen);
                              ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                              ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                              ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                              ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                              ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                              ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                              ObjectSetInteger(ChartID(),"Long&Short",OBJPROP_BGCOLOR,clrWhiteSmoke);
                              ObjectSetInteger(ChartID(),"Long&Short",OBJPROP_STATE,false);
                              ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_STATE,false);
                              ChartSetInteger(ChartID(),CHART_SHOW_TRADE_LEVELS,false);
                              ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_BGCOLOR,clrWhiteSmoke);


                              ObjectDelete(0,"SLHLine1");
                              ObjectDelete(0,"TPHLine1");
                              ObjectDelete(0,"SLHLine3");

                              HLineCreate(0,"SLHLine2",SLPoint,"sell");
                              HLineCreate(0,"TPHLine2",-(SLPoint+SL_Space)*RatioTPSL,"sell");

                              HLineCreate(0,"SLHLine4",-SL_Space,"sell");
                              ObjectSetInteger(ChartID(),"SLHLine4",OBJPROP_COLOR,clrWhite);

                              ObjectSetInteger(ChartID(),"TPHLine2",OBJPROP_COLOR,clrLightGreen);
                              ObjectSetInteger(ChartID(),"TPHLine2",OBJPROP_WIDTH,2);

                             }
                           else
                             {

                              ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                              ObjectDelete(0,"SLHLine2");
                              ObjectDelete(0,"TPHLine2");
                              ObjectDelete(0,"SLHLine4");

                             }
                       }
                     else
                        if(sparam=="BuyButton")
                          {

                           if(PositionsTotal()+OrdersTotal() >=MaxPositions)
                              MessageBox("Can not open new positions due to Max_positions","Maximun positions Message",0);
                           if(PositionsTotal()+OrdersTotal() < MaxPositions)
                              if(CurrentMarginLevel>=MarginLevelLimit+50||PositionsTotal()==0)
                                 if(PositionsTotal()<MaxPositions)
                                    if(SymbolInfoInteger(_Symbol,SYMBOL_SPREAD)<StringToInteger(ObjectGetString(ChartID(),"SpreadMax",OBJPROP_TEXT)))
                                      {
                                       Comment(sparam+" Was Pressed  "+(string)GetLastError());
                                       if(Tradingpositontype=="Long"|| Tradingpositontype=="Long&Short")
                                         {
                                          double SL_HLine_price1=ObjectGetDouble(0,"SLHLine1",OBJPROP_PRICE);
                                          int SL_Price1=(Ask-SL_HLine_price1)/_Point;
                                          double TP_HLine_Price1=ObjectGetDouble(0,"TPHLine1",OBJPROP_PRICE);
                                          int TP_Price1=(-Ask+TP_HLine_Price1)/_Point;

                                          int buy_confirm=MessageBox("WARING! BuyLimint "
                                                                     +"\n"+ ""
                                                                     +"\n"+ "Currency pair:  "  + _Symbol
                                                                     +"\n"+ "Lot size:       "  + Lot_sizeTotrade


                                                                     +"\n"+ "Used margin:    "  + int (CalculateUsedMarginForNewPosition(Lot_sizeTotrade,_Symbol))    +" USD"
                                                                     +"\n"+ "Margin level:   "  + int (CalculateMarginLevelForNewPosition(Lot_sizeTotrade,_Symbol))   +" %"
                                                                     +"\n"+ "Stop Loss:      "  + SL_Price1                  +" Point" + " "+ int(CalculatePointValue(_Symbol)*SL_Price1*Lot_sizeTotrade)+" USD"
                                                                     +"\n"+ ""
                                                                     +"\n"+ ""
                                                                     +"\n"+ " Do you want to open new position ?","New position Message",MB_OKCANCEL);

                                          if(buy_confirm==1)
                                            {
                                             trade.Buy(Lot_sizeTotrade,_Symbol,Ask,NormalizeDouble(Ask-_Point*SL_Price1,_Digits),NormalizeDouble(Ask+_Point*TP_Price1,_Digits),TradingTimeFrameComment);
                                             ObjectSetInteger(0,"LongOnly",OBJPROP_STATE,false);
                                             ObjectSetInteger(0,"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                             ObjectDelete(0,"SLHLine1");
                                             ObjectDelete(0,"TPHLine1");
                                             ObjectSetInteger(0,"ShowTradeLevels",OBJPROP_STATE,true);
                                             ChartSetInteger(0,CHART_SHOW_TRADE_LEVELS,true);
                                             ObjectSetInteger(0,"ShowTradeLevels",OBJPROP_BGCOLOR,clrLightGreen);
                                            }
                                         }
                                       SetTimeCount=TimeGMT();
                                      }
                                    else
                                       Comment("Robort"+" Can't Open Long Position dueto margin level lower or moving average condition");

                           SetTimeCount=TimeGMT();
                           ObjectSetInteger(ChartID(),"BuyButton",OBJPROP_STATE,false);

                          }

                        else
                           if(sparam=="SellButton")
                             {
                              if(PositionsTotal()+OrdersTotal() >= MaxPositions)
                                 MessageBox("Can not open new positions due to Max_positions","Maximun positions Message",0);
                              if(PositionsTotal()+OrdersTotal() <MaxPositions)
                                 if(Lot_sizeTotrade-AllowedLotsize<=0)
                                    if(CurrentMarginLevel>=MarginLevelLimit+50||PositionsTotal()==0)
                                       if(PositionsTotal()<MaxPositions)
                                          if(SymbolInfoInteger(_Symbol,SYMBOL_SPREAD)<StringToInteger(ObjectGetString(ChartID(),"SpreadMax",OBJPROP_TEXT)))
                                            {
                                             Comment(sparam+" Was Pressed");
                                             if(Tradingpositontype=="Short"|| Tradingpositontype=="Long&Short")
                                               {

                                                double SL_HLine_price2=ObjectGetDouble(0,"SLHLine2",OBJPROP_PRICE);
                                                int SL_Price2=(-Bid+SL_HLine_price2)/_Point;
                                                double TP_HLine_Price2=ObjectGetDouble(0,"TPHLine2",OBJPROP_PRICE);
                                                int TP_Price2=(Bid-TP_HLine_Price2)/_Point;





                                                int Sell_confirm=MessageBox("WARING! SellLimit "
                                                                            +"\n"+ ""
                                                                            +"\n"+ "Currency pair:  "  + _Symbol
                                                                            +"\n"+ "Lot size:       "  + Lot_sizeTotrade


                                                                            +"\n"+ "Used margin:    "  + int (CalculateUsedMarginForNewPosition(Lot_sizeTotrade,_Symbol))    +" USD"
                                                                            +"\n"+ "Margin level:   "  + int (CalculateMarginLevelForNewPosition(Lot_sizeTotrade,_Symbol))   +" %"
                                                                            +"\n"+ "Stop Loss:      "  + SL_Price2                   +" Point" + " "+ int(CalculatePointValue(_Symbol)*SL_Price2*Lot_sizeTotrade)+" USD"
                                                                            +"\n"+ ""
                                                                            +"\n"+ ""
                                                                            +"\n"+ " Do you want to open new position ?","New position Message",MB_OKCANCEL);




                                                if(Sell_confirm==1)
                                                  {
                                                   trade.Sell(Lot_sizeTotrade,_Symbol,Bid,NormalizeDouble(Bid+_Point*SL_Price2,_Digits),NormalizeDouble(Bid-_Point*TP_Price2,_Digits),TradingTimeFrameComment);

                                                   ObjectSetInteger(0,"ShortOnly",OBJPROP_STATE,false);
                                                   ObjectSetInteger(0,"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                   ObjectDelete(0,"SLHLine2");
                                                   ObjectDelete(0,"TPHLine2");
                                                   ObjectSetInteger(0,"ShowTradeLevels",OBJPROP_STATE,true);
                                                   ChartSetInteger(0,CHART_SHOW_TRADE_LEVELS,true);
                                                   ObjectSetInteger(0,"ShowTradeLevels",OBJPROP_BGCOLOR,clrLightGreen);
                                                  }

                                               }
                                             SetTimeCount=TimeGMT();
                                            }
                                          else
                                             Comment("Rorbot"+" Can't Open Short Position dueto margin level lower or moving average condition");
                              SetTimeCount=TimeGMT();
                              ObjectSetInteger(ChartID(),"SellButton",OBJPROP_STATE,false);
                             }

                           else
                              if(sparam=="BuyStop")
                                {

                                 if(PositionsTotal() +OrdersTotal() >= MaxPositions)
                                    MessageBox("Can not open new positions due to Max_positions","Maximun positions Message",0);

                                 if(PositionsTotal()+OrdersTotal() <MaxPositions)
                                    if(Lot_sizeTotrade-AllowedLotsize<=0)
                                       if(PositionsTotal()<MaxPositions)
                                          if(Tradingpositontype=="Buy_Stop")
                                            {

                                             double SL_HLine_price1=ObjectGetDouble(0,"SLHLine1",OBJPROP_PRICE);
                                             int SL_Price1=(Ask-SL_HLine_price1)/_Point;
                                             double TP_HLine_Price1=ObjectGetDouble(0,"TPHLine1",OBJPROP_PRICE);
                                             int TP_Price1=(-Ask+TP_HLine_Price1)/_Point;

                                             int BuyStop_confrim=MessageBox("WARING! BuyStop"
                                                                            +"\n"+ ""
                                                                            +"\n"+ "Currency pair:  "  + _Symbol
                                                                            +"\n"+ "Lot size:       "  + Lot_sizeTotrade


                                                                            +"\n"+ "Used margin:    "  + int (CalculateUsedMarginForNewPosition(Lot_sizeTotrade,_Symbol))    +" USD"
                                                                            +"\n"+ "Margin level:   "  + int (CalculateMarginLevelForNewPosition(Lot_sizeTotrade,_Symbol))   +" %"
                                                                            +"\n"+ "Stop Loss:      "  + SL_Price1                    +" Point" + " "+  int(CalculatePointValue(_Symbol)*SL_Price1*Lot_sizeTotrade)+" USD"
                                                                            +"\n"+ ""
                                                                            +"\n"+ ""
                                                                            +"\n"+ " Do you want to open new position ?","New position Message",MB_OKCANCEL);




                                             if(BuyStop_confrim==1)
                                               {
                                                trade.BuyStop(Lot_sizeTotrade,ObjectGetDouble(ChartID(),"SLHLine3",OBJPROP_PRICE),_Symbol,NormalizeDouble(Ask-_Point*SL_Price1,_Digits),NormalizeDouble(Ask+_Point*TP_Price1,_Digits),0,0,TradingTimeFrameComment);
                                                ObjectSetInteger(0,"BuyStopSelect",OBJPROP_STATE,false);
                                                ObjectSetInteger(0,"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                ObjectDelete(0,"SLHLine1");
                                                ObjectDelete(0,"TPHLine1");
                                                ObjectDelete(0,"SLHLine3");
                                                ObjectSetInteger(0,"ShowTradeLevels",OBJPROP_STATE,true);
                                                ChartSetInteger(0,CHART_SHOW_TRADE_LEVELS,true);




                                               }
                                             SetTimeCount=TimeGMT();
                                            }

                                          else
                                             Comment("Robort Can't Place A BuyStop dueto margin level lower or moving average condition");
                                 SetTimeCount=TimeGMT();
                                 ObjectSetInteger(ChartID(),"BuyStop",OBJPROP_STATE,false);

                                }

                              else
                                 if(sparam=="SellStop")
                                   {

                                    if(PositionsTotal() +OrdersTotal() >=MaxPositions)
                                       MessageBox("Can not open new positions due to Max_positions","Maximun positions Message",0);
                                    if(PositionsTotal()+OrdersTotal() < MaxPositions)
                                       if(Lot_sizeTotrade-AllowedLotsize<=0)
                                          if(PositionsTotal()<MaxPositions)
                                             if(Tradingpositontype=="Sell_Stop")
                                               {

                                                double SL_HLine_price2=ObjectGetDouble(0,"SLHLine2",OBJPROP_PRICE);
                                                int SL_Price2=(-Bid+SL_HLine_price2)/_Point;
                                                double TP_HLine_Price2=ObjectGetDouble(0,"TPHLine2",OBJPROP_PRICE);
                                                int TP_Price2=(Bid-TP_HLine_Price2)/_Point;

                                                int SellStop_confirm=MessageBox("WARING! SellStop "
                                                                                +"\n"+ ""
                                                                                +"\n"+ "Currency pair:  "  + _Symbol
                                                                                +"\n"+ "Lot size:       "  + Lot_sizeTotrade


                                                                                +"\n"+ "Used margin:    "  + int (CalculateUsedMarginForNewPosition(Lot_sizeTotrade,_Symbol))    +" USD"
                                                                                +"\n"+ "Margin level:   "  + int (CalculateMarginLevelForNewPosition(Lot_sizeTotrade,_Symbol))   +" %"
                                                                                +"\n"+ "Stop Loss:      "  + SL_Price2                    +" Point" + " "+ int(CalculatePointValue(_Symbol)*SL_Price2*Lot_sizeTotrade)+" USD"
                                                                                +"\n"+ ""
                                                                                +"\n"+ ""
                                                                                +"\n"+ " Do you want to open new position ?","New position Message",MB_OKCANCEL);


                                                if(SellStop_confirm==1)
                                                  {
                                                   trade.SellStop(Lot_sizeTotrade,ObjectGetDouble(ChartID(),"SLHLine4",OBJPROP_PRICE),_Symbol,NormalizeDouble(Bid+_Point*SL_Price2,_Digits),NormalizeDouble(Bid-_Point*TP_Price2,_Digits),0,0,TradingTimeFrameComment);
                                                   ObjectSetInteger(0,"SellStopSelect",OBJPROP_STATE,false);
                                                   ObjectSetInteger(0,"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                   ObjectDelete(0,"SLHLine2");
                                                   ObjectDelete(0,"SLHLine4");
                                                   ObjectDelete(0,"TPHLine2");
                                                   ObjectSetInteger(0,"ShowTradeLevels",OBJPROP_STATE,true);
                                                   ChartSetInteger(0,CHART_SHOW_TRADE_LEVELS,true);


                                                  }
                                                SetTimeCount=TimeGMT();


                                               }

                                             else
                                                Comment("Robort Can't Place A SellStop dueto margin level lower or moving average condition");
                                    SetTimeCount=TimeGMT();
                                    ObjectSetInteger(ChartID(),"SellStop",OBJPROP_STATE,false);

                                   }

                                 else
                                    if(sparam=="CancelOrderStop")
                                      {
                                       Comment("All Pending Orders Are Cancelled");
                                       SetTimeCount=TimeGMT();
                                       for(int i=OrdersTotal()-1; i>=0; i--)
                                         {
                                          ulong Orderticket=OrderGetTicket(i);
                                          string symbol=OrderGetString(ORDER_SYMBOL);
                                          if(OrderGetInteger(ORDER_TYPE)==ORDER_TYPE_BUY_STOP||OrderGetInteger(ORDER_TYPE)==ORDER_TYPE_SELL_STOP||
                                             OrderGetInteger(ORDER_TYPE)==ORDER_TYPE_SELL_LIMIT||OrderGetInteger(ORDER_TYPE)==ORDER_TYPE_BUY_LIMIT)
                                             if(symbol==_Symbol)
                                                trade.OrderDelete(Orderticket);
                                         }

                                       ObjectSetInteger(ChartID(),"CancelOrderStop",OBJPROP_STATE,false);

                                      }

                                    else
                                       if(sparam=="Reset_initialBalance")
                                         {

                                          int Refresh_InitialAccount=MessageBox("Do you want to refresh initial account ??","Initial Account Message",MB_OKCANCEL);
                                          if(Refresh_InitialAccount==1)
                                            {
                                             InitialBalance=NormalizeDouble(AccountInfoDouble(ACCOUNT_BALANCE),2);
                                             ObjectSetInteger(ChartID(),"Reset_initialBalance",OBJPROP_STATE,false);
                                            }
                                         }
                                       else
                                          if(sparam=="Object_study")
                                            {
                                             ObjectsDeleteAll(ChartID(),0,OBJ_TREND);
                                             ObjectsDeleteAll(ChartID(),0,OBJ_CHANNEL);
                                             ObjectsDeleteAll(ChartID(),0,OBJ_VLINE);
                                             ObjectsDeleteAll(ChartID(),0,OBJ_HLINE);
                                             ObjectsDeleteAll(ChartID(),1,OBJ_TREND);
                                             ObjectsDeleteAll(ChartID(),1,OBJ_CHANNEL);
                                             ObjectsDeleteAll(ChartID(),1,OBJ_VLINE);
                                             ObjectsDeleteAll(ChartID(),1,OBJ_HLINE);
                                             ObjectsDeleteAll(ChartID(),0,OBJ_RECTANGLE);
                                             ObjectsDeleteAll(ChartID(),0,OBJ_REGRESSION);

                                             ObjectSetInteger(ChartID(),"Object_study",OBJPROP_STATE,false);
                                             ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);
                                             ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);

                                             ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);
                                             ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);

                                             ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);
                                             ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                             ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                             ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);

                                            }
                                          else
                                             if(sparam=="RSI_Moment")
                                               {

                                                ChartIndicatorDelete(ChartID(),1,"DIEN_TRIX R0");
                                                ObjectSetInteger(ChartID(),"TRIX",OBJPROP_STATE,false);
                                                ObjectSetInteger(ChartID(),"TRIX",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                bool RSI_Momenttate=false;
                                                RSI_Momenttate=ObjectGetInteger(ChartID(),"RSI_Moment",OBJPROP_STATE);
                                                int RSI=iCustom(_Symbol,PERIOD_CURRENT,"Dien\\Dien_RSI+Moving Average R1");
                                                int Moment=iCustom(_Symbol,PERIOD_CURRENT,"Dien\\Dien_Moment+Moving Average R1");

                                                if(RSI_Momenttate)
                                                  {
                                                   ChartIndicatorDelete(ChartID(),1,"Dien_Moment+Moving Average R1");
                                                   ChartIndicatorAdd(ChartID(),1,RSI);
                                                   ObjectSetString(ChartID(),"RSI_Moment",OBJPROP_TEXT,"RSI");
                                                   ObjectSetInteger(ChartID(),"RSI_Moment",OBJPROP_BGCOLOR,clrLightGreen);

                                                  }
                                                else
                                                  {
                                                   ChartIndicatorDelete(ChartID(),1,"Dien_RSI+Moving Average R1");
                                                   ChartIndicatorAdd(ChartID(),1,Moment);
                                                   ObjectSetString(ChartID(),"RSI_Moment",OBJPROP_TEXT,"Moment");
                                                   ObjectSetInteger(ChartID(),"RSI_Moment",OBJPROP_BGCOLOR,clrLightGreen);

                                                  }

                                               }

                                             else
                                                if(sparam=="TRIX")
                                                  {

                                                   bool TRIXState=false;
                                                   TRIXState=ObjectGetInteger(ChartID(),"TRIX",OBJPROP_STATE);
                                                   int TRIX=iCustom(_Symbol,PERIOD_CURRENT,"Dien\\DIEN_TRIX R0");

                                                   if(TRIXState)
                                                     {

                                                      ChartIndicatorAdd(ChartID(),1,TRIX);
                                                      ChartIndicatorDelete(ChartID(),1,"Dien_RSI+Moving Average R1");
                                                      ChartIndicatorDelete(ChartID(),1,"Dien_Moment+Moving Average R1");
                                                      ObjectSetInteger(ChartID(),"TRIX",OBJPROP_BGCOLOR,clrLightGreen);

                                                     }
                                                   else
                                                     {
                                                      ChartIndicatorDelete(ChartID(),1,"DIEN_TRIX R0");
                                                      ObjectSetInteger(ChartID(),"TRIX",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                      if(ObjectGetString(ChartID(),"RSI_Moment",OBJPROP_TEXT)=="RSI")
                                                         ChartIndicatorAdd(ChartID(),1,iCustom(_Symbol,PERIOD_CURRENT,"Dien\\Dien_RSI+Moving Average R1"));
                                                      else
                                                         ChartIndicatorAdd(ChartID(),1,iCustom(_Symbol,PERIOD_CURRENT,"Dien\\Dien_Moment+Moving Average R1"));

                                                     }

                                                  }

                                                else
                                                   if(sparam=="ZigZag")
                                                     {

                                                      bool ZigZagState=false;
                                                      ZigZagState=ObjectGetInteger(ChartID(),"ZigZag",OBJPROP_STATE);
                                                      int zizag=iCustom(_Symbol,PERIOD_CURRENT,"Dien\\Dien_ZigzagColor");

                                                      if(ZigZagState)
                                                        {

                                                         ChartSetInteger(ChartID(),CHART_MODE,CHART_CANDLES);
                                                         ChartIndicatorAdd(ChartID(),0,zizag);
                                                         ObjectSetInteger(ChartID(),"ZigZag",OBJPROP_BGCOLOR,clrLightGreen);
                                                         ChartIndicatorDelete(ChartID(),0,"Dien_Price+MA R3");
                                                         ChartIndicatorDelete(ChartID(),0,"Dien_Price+MA R4");
                                                         ObjectSetInteger(ChartID(),"MovingAverage",OBJPROP_STATE,false);
                                                         ObjectSetInteger(ChartID(),"MovingAverage",OBJPROP_BGCOLOR,clrWhiteSmoke);


                                                        }
                                                      else
                                                        {
                                                         ChartIndicatorDelete(ChartID(),0,"ZigZagColor(12,5,3)");
                                                         ObjectSetInteger(ChartID(),"ZigZag",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                         //if(!ObjectGetInteger(ChartID(),"MovingAverage",OBJPROP_STATE))
                                                         // ChartSetInteger(ChartID(),CHART_MODE,CHART_BARS);

                                                        }

                                                     }

                                                   else
                                                      if(sparam=="MovingAverage")
                                                        {

                                                         bool MAState=false;
                                                         MAState=ObjectGetInteger(ChartID(),"MovingAverage",OBJPROP_STATE);

                                                         if(MAState)
                                                           {
                                                            ObjectSetString(ChartID(),"MovingAverage",OBJPROP_TEXT,"Candle");
                                                            ChartIndicatorDelete(ChartID(),0,"ZigZagColor(12,5,3)");
                                                            ChartIndicatorDelete(ChartID(),0,"Dien_Price+MA R4");
                                                            ChartSetInteger(ChartID(),CHART_MODE,CHART_LINE);
                                                            color ChartLine_color=ChartGetInteger(ChartID(),CHART_COLOR_BACKGROUND);
                                                            ChartSetInteger(ChartID(),CHART_COLOR_CHART_LINE,ChartLine_color);

                                                            int MA3=iCustom(_Symbol,PERIOD_CURRENT,"Dien\\Dien_Price+MA R3");
                                                            ChartIndicatorAdd(ChartID(),0,MA3);

                                                            ObjectSetInteger(ChartID(),"MovingAverage",OBJPROP_BGCOLOR,clrLightGreen);

                                                            ObjectSetInteger(ChartID(),"ZigZag",OBJPROP_STATE,false);
                                                            ObjectSetInteger(ChartID(),"ZigZag",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                           }
                                                         else
                                                           {

                                                            ObjectSetString(ChartID(),"MovingAverage",OBJPROP_TEXT,"Line");
                                                            ChartIndicatorDelete(ChartID(),0,"Dien_Price+MA R3");
                                                            int MA4=iCustom(_Symbol,PERIOD_CURRENT,"Dien\\Dien_Price+MA R4");
                                                            ChartIndicatorAdd(ChartID(),0,MA4);
                                                            ChartSetInteger(ChartID(),CHART_MODE,CHART_LINE);
                                                            ChartSetInteger(ChartID(),CHART_COLOR_CHART_LINE,clrWhiteSmoke);
                                                            ObjectSetInteger(ChartID(),"MovingAverage",OBJPROP_BGCOLOR,clrRed);
                                                            //ChartSetInteger(ChartID(),CHART_MODE,CHART_CANDLES);
                                                           }
                                                        }


                                                      else
                                                         if(sparam=="ShowTradeLevels")
                                                           {
                                                            bool ShowTradeLevelsState=false;
                                                            ShowTradeLevelsState=ObjectGetInteger(ChartID(),"ShowTradeLevels",OBJPROP_STATE);
                                                            if(ShowTradeLevelsState)
                                                              {
                                                               ChartSetInteger(ChartID(),CHART_SHOW_TRADE_LEVELS,true);
                                                               ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_BGCOLOR,clrLightGreen);
                                                              }
                                                            else
                                                              {
                                                               ChartSetInteger(ChartID(),CHART_SHOW_TRADE_LEVELS,false);
                                                               ObjectSetInteger(ChartID(),"ShowTradeLevels",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                              }
                                                           }

                                                         else
                                                            if(sparam=="TakeProfit")
                                                              {

                                                               for(int i=PositionsTotal(); i>=0; i--)
                                                                 {
                                                                  ulong PositionTicket=PositionGetTicket(i);
                                                                  string symbol=PositionGetSymbol(i);
                                                                  if(symbol==_Symbol)
                                                                     if(PositionGetDouble(POSITION_PROFIT)>=0)
                                                                       {
                                                                        trade.PositionClose(PositionTicket,0);
                                                                        //break;
                                                                       }
                                                                 }
                                                               ObjectSetInteger(ChartID(),"TakeProfit",OBJPROP_STATE,false);
                                                              }

                                                            else
                                                               if(sparam=="StopLoss")
                                                                 {
                                                                  for(int i=PositionsTotal(); i>=0; i--)
                                                                    {
                                                                     ulong PositionTicket=PositionGetTicket(i);
                                                                     double Position_profit=NormalizeDouble(PositionGetDouble(POSITION_PROFIT),2);
                                                                     string symbol=PositionGetSymbol(i);
                                                                     if(symbol==_Symbol)
                                                                        if(PositionGetDouble(POSITION_PROFIT)<=0)
                                                                          {
                                                                           int StopLoss_confirm=MessageBox("Do you want to stop loss "+symbol+" Ticket No: "+PositionTicket+" with loss: "+ Position_profit+" ??","StopLoss Message",MB_OKCANCEL);
                                                                           if(StopLoss_confirm==1)
                                                                             {
                                                                              trade.PositionClose(PositionTicket,0);
                                                                              break;
                                                                             }
                                                                          }
                                                                    }

                                                                  ObjectSetInteger(ChartID(),"StopLoss",OBJPROP_STATE,false);

                                                                 }

                                                               else
                                                                  if(sparam=="Stop1/3")
                                                                    {

                                                                     for(int i=PositionsTotal(); i>=0; i--)
                                                                       {
                                                                        ulong PositionTicket=PositionGetTicket(i);
                                                                        double Position_profit=NormalizeDouble(1/3*PositionGetDouble(POSITION_PROFIT),2);
                                                                        string symbol=PositionGetSymbol(i);

                                                                        if(symbol==_Symbol)
                                                                           if(PositionGetDouble(POSITION_PROFIT)<=0)
                                                                             {


                                                                              double PositionVolume=PositionGetDouble(POSITION_VOLUME);
                                                                              int Stoploss13_confirm=MessageBox("Do you want to stop loss "+symbol+" Ticket No: "+PositionTicket+" with loss: "+ Position_profit+" ??","StopLoss Message",MB_OKCANCEL);
                                                                              if(Stoploss13_confirm==1)
                                                                                {
                                                                                 trade.PositionClosePartial(PositionTicket,NormalizeDouble(PositionVolume/3,2),0);
                                                                                 break;
                                                                                }
                                                                             }
                                                                       }
                                                                     ObjectSetInteger(ChartID(),"Stop1/3",OBJPROP_STATE,false);
                                                                    }

                                                                  else
                                                                     if(sparam=="ZeroLoss")
                                                                       {

                                                                        for(int i=PositionsTotal(); i>=0; i--)
                                                                          {
                                                                           ulong PositionTicket=PositionGetTicket(i);
                                                                           string symbol=PositionGetSymbol(i);


                                                                           if(symbol==_Symbol)
                                                                              if(PositionGetDouble(POSITION_PROFIT)>0)
                                                                                {
                                                                                 double Point_Value=SymbolInfoDouble(symbol,SYMBOL_POINT);
                                                                                 int DigitalValue=(int)SymbolInfoInteger(symbol,SYMBOL_DIGITS);
                                                                                 if(PositionGetInteger(POSITION_TYPE)==POSITION_TYPE_BUY)
                                                                                    trade.PositionModify(PositionTicket,NormalizeDouble(PositionGetDouble(POSITION_PRICE_OPEN)+10*Point_Value,DigitalValue),PositionGetDouble(POSITION_TP));
                                                                                 else
                                                                                    trade.PositionModify(PositionTicket,NormalizeDouble(PositionGetDouble(POSITION_PRICE_OPEN)-10*Point_Value,DigitalValue),PositionGetDouble(POSITION_TP));

                                                                                 break;
                                                                                }
                                                                          }
                                                                        ObjectSetInteger(ChartID(),"ZeroLoss",OBJPROP_STATE,false);
                                                                       }

                                                                     else
                                                                        if(sparam=="TakehalfofProfit")
                                                                          {

                                                                           for(int i=PositionsTotal(); i>=0; i--)
                                                                             {
                                                                              ulong PositionTicket=PositionGetTicket(i);
                                                                              string symbol=PositionGetSymbol(i);

                                                                              if(symbol==_Symbol)
                                                                                 if(PositionGetDouble(POSITION_PROFIT)>0)
                                                                                   {

                                                                                    double PositionVolume=PositionGetDouble(POSITION_VOLUME);
                                                                                    trade.PositionClosePartial(PositionTicket,NormalizeDouble(PositionVolume/2,2),0);
                                                                                    break;
                                                                                   }

                                                                             }
                                                                           ObjectSetInteger(ChartID(),"TakehalfofProfit",OBJPROP_STATE,false);
                                                                          }


                                                                        else
                                                                           if(sparam=="CompleteTarget")
                                                                             {
                                                                              MessageBox(" Do you want to complete traiding ??","Trading completion Message");
                                                                              for(int i=PositionsTotal(); i>=0; i--)
                                                                                {
                                                                                 ulong PositionTicket=PositionGetTicket(i);
                                                                                 string symbol=PositionGetSymbol(i);

                                                                                 if(PositionGetDouble(POSITION_PROFIT)>0)
                                                                                   {
                                                                                    trade.PositionClose(PositionTicket,0);

                                                                                   }

                                                                                }

                                                                              ObjectSetInteger(ChartID(),"CompleteTarget",OBJPROP_STATE,false);

                                                                             }
                                                                           else
                                                                              if(sparam=="NormalTrailingStop")
                                                                                {

                                                                                 TrailingStopSelection="";

                                                                                 bool NormalTrailingstopState=false;
                                                                                 NormalTrailingstopState=ObjectGetInteger(ChartID(),"NormalTrailingStop",OBJPROP_STATE);


                                                                                 if(NormalTrailingstopState)
                                                                                   {
                                                                                    ObjectSetInteger(ChartID(),"NormalTrailingStop",OBJPROP_BGCOLOR,clrLightGreen);
                                                                                    ObjectSetInteger(ChartID(),"FastTrailingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                    ObjectSetInteger(ChartID(),"FastTrailingStop",OBJPROP_STATE,false);
                                                                                    ObjectSetInteger(ChartID(),"DynamicTraingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                    ObjectSetInteger(ChartID(),"DynamicTraingStop",OBJPROP_STATE,false);

                                                                                   }
                                                                                 else
                                                                                   {

                                                                                    ObjectSetInteger(ChartID(),"NormalTrailingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                   }

                                                                                }
                                                                              else
                                                                                 if(sparam=="FastTrailingStop")
                                                                                   {


                                                                                    bool FastTrailingstopState=false;
                                                                                    FastTrailingstopState=ObjectGetInteger(ChartID(),"FastTrailingStop",OBJPROP_STATE);

                                                                                    if(FastTrailingstopState)
                                                                                      {
                                                                                       ChartSetSymbolPeriod(ChartID(),_Symbol,PERIOD_M1);
                                                                                       ObjectDelete(ChartID(),"TIMEFRAMEValue");
                                                                                       ObjectSetString(ChartID(),"TradingTimeframeButton",OBJPROP_TEXT,"M1");



                                                                                       ObjectSetInteger(ChartID(),"FastTrailingStop",OBJPROP_BGCOLOR,clrLightGreen);
                                                                                       ObjectSetInteger(ChartID(),"NormalTrailingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                       ObjectSetInteger(ChartID(),"NormalTrailingStop",OBJPROP_STATE,false);
                                                                                       ObjectSetInteger(ChartID(),"DynamicTraingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                       ObjectSetInteger(ChartID(),"DynamicTraingStop",OBJPROP_STATE,false);

                                                                                      }
                                                                                    else
                                                                                      {

                                                                                       ObjectSetInteger(ChartID(),"FastTrailingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                       ChartSetSymbolPeriod(ChartID(),_Symbol,TradingTimeframe);
                                                                                       ObjectDelete(ChartID(),"TIMEFRAMEValue");
                                                                                       ObjectSetString(ChartID(),"TradingTimeframeButton",OBJPROP_TEXT,getstringoftimeframe(TradingTimeframe));

                                                                                      }


                                                                                   }

                                                                                 else
                                                                                    if(sparam=="DynamicTraingStop")
                                                                                      {

                                                                                       ChartSetSymbolPeriod(ChartID(),_Symbol,TradingTimeframe);

                                                                                       bool DynamicTrailingstopState=false;
                                                                                       DynamicTrailingstopState=ObjectGetInteger(ChartID(),"DynamicTraingStop",OBJPROP_STATE);

                                                                                       if(DynamicTrailingstopState)
                                                                                         {
                                                                                          ObjectSetInteger(ChartID(),"DynamicTraingStop",OBJPROP_BGCOLOR,clrLightGreen);
                                                                                          ObjectSetInteger(ChartID(),"NormalTrailingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                          ObjectSetInteger(ChartID(),"NormalTrailingStop",OBJPROP_STATE,false);
                                                                                          ObjectSetInteger(ChartID(),"FastTrailingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                          ObjectSetInteger(ChartID(),"FastTrailingStop",OBJPROP_STATE,false);
                                                                                         }
                                                                                       else
                                                                                         {
                                                                                          ObjectSetInteger(ChartID(),"DynamicTraingStop",OBJPROP_BGCOLOR,clrWhiteSmoke);

                                                                                         }

                                                                                      }

                                                                                    else
                                                                                       if(sparam=="MaxSLValue")
                                                                                         {
                                                                                          ObjectSetString(ChartID(),"SLValue",OBJPROP_TEXT,IntegerToString(CalculateMaxSLPoints(_Symbol,RiskAccount,SLPoint)-75));

                                                                                         }
                                                                                       else
                                                                                          if(sparam=="AUDCHF")
                                                                                            {
                                                                                             ChartSetSymbolPeriod(ChartID(),"AUDCHF",_Period);
                                                                                             ObjectSetInteger(ChartID(),"AUDCHF",OBJPROP_STATE,false);
                                                                                             ObjectSetInteger(ChartID(),"AUDCHF",OBJPROP_BGCOLOR,clrGreen);

                                                                                             ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                             ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                             ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                             ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                             ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                             ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                             ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                             ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                             ObjectDelete(0,"SLHLine2");
                                                                                             ObjectDelete(0,"SLHLine1");
                                                                                             ObjectDelete(0,"TPHLine1");
                                                                                             ObjectDelete(0,"TPHLine2");
                                                                                             ObjectDelete(0,"SLHLine3");
                                                                                             ObjectDelete(0,"SLHLine4");
                                                                                             ObjectDelete(0,"Text_SL_HLine1");
                                                                                             ObjectDelete(0,"Text_TP_HLine1");
                                                                                             ObjectDelete(0,"Text_SL_HLine2");
                                                                                             ObjectDelete(0,"Text_TP_HLine2");


                                                                                            }


                                                                                          else
                                                                                             if(sparam=="AUDJPY")
                                                                                               {
                                                                                                ChartSetSymbolPeriod(ChartID(),"AUDJPY",_Period);
                                                                                                ObjectSetInteger(ChartID(),"AUDJPY",OBJPROP_STATE,false);
                                                                                                ObjectSetInteger(ChartID(),"AUDJPY",OBJPROP_BGCOLOR,clrGreen);

                                                                                                ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                ObjectDelete(0,"SLHLine2");
                                                                                                ObjectDelete(0,"SLHLine1");
                                                                                                ObjectDelete(0,"TPHLine1");
                                                                                                ObjectDelete(0,"TPHLine2");
                                                                                                ObjectDelete(0,"SLHLine3");
                                                                                                ObjectDelete(0,"SLHLine4");
                                                                                                ObjectDelete(0,"Text_SL_HLine1");
                                                                                                ObjectDelete(0,"Text_TP_HLine1");
                                                                                                ObjectDelete(0,"Text_SL_HLine2");
                                                                                                ObjectDelete(0,"Text_TP_HLine2");

                                                                                               }

                                                                                             else
                                                                                                if(sparam=="AUDNZD")
                                                                                                  {
                                                                                                   ChartSetSymbolPeriod(ChartID(),"AUDNZD",_Period);
                                                                                                   ObjectSetInteger(ChartID(),"AUDNZD",OBJPROP_STATE,false);
                                                                                                   ObjectSetInteger(ChartID(),"AUDNZD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                   ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                   ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                   ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                   ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                   ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                   ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                   ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                   ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                   ObjectDelete(0,"SLHLine2");
                                                                                                   ObjectDelete(0,"SLHLine1");
                                                                                                   ObjectDelete(0,"TPHLine1");
                                                                                                   ObjectDelete(0,"TPHLine2");
                                                                                                   ObjectDelete(0,"SLHLine3");
                                                                                                   ObjectDelete(0,"SLHLine4");
                                                                                                   ObjectDelete(0,"Text_SL_HLine1");
                                                                                                   ObjectDelete(0,"Text_TP_HLine1");
                                                                                                   ObjectDelete(0,"Text_SL_HLine2");
                                                                                                   ObjectDelete(0,"Text_TP_HLine2");

                                                                                                  }

                                                                                                else
                                                                                                   if(sparam=="AUDUSD")
                                                                                                     {
                                                                                                      ChartSetSymbolPeriod(ChartID(),"AUDUSD",_Period);
                                                                                                      ObjectSetInteger(ChartID(),"AUDUSD",OBJPROP_STATE,false);
                                                                                                      ObjectSetInteger(ChartID(),"AUDUSD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                      ObjectDelete(0,"SLHLine2");
                                                                                                      ObjectDelete(0,"SLHLine1");
                                                                                                      ObjectDelete(0,"TPHLine1");
                                                                                                      ObjectDelete(0,"TPHLine2");
                                                                                                      ObjectDelete(0,"SLHLine3");
                                                                                                      ObjectDelete(0,"SLHLine4");
                                                                                                      ObjectDelete(0,"Text_SL_HLine1");
                                                                                                      ObjectDelete(0,"Text_TP_HLine1");
                                                                                                      ObjectDelete(0,"Text_SL_HLine2");
                                                                                                      ObjectDelete(0,"Text_TP_HLine2");

                                                                                                     }
                                                                                                   else
                                                                                                      if(sparam=="CADJPY")
                                                                                                        {
                                                                                                         ChartSetSymbolPeriod(ChartID(),"CADJPY",_Period);
                                                                                                         ObjectSetInteger(ChartID(),"CADJPY",OBJPROP_STATE,false);
                                                                                                         ObjectSetInteger(ChartID(),"CADJPY",OBJPROP_BGCOLOR,clrGreen);

                                                                                                         ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                         ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                         ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                         ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                         ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                         ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                         ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                         ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                         ObjectDelete(0,"SLHLine2");
                                                                                                         ObjectDelete(0,"SLHLine1");
                                                                                                         ObjectDelete(0,"TPHLine1");
                                                                                                         ObjectDelete(0,"TPHLine2");
                                                                                                         ObjectDelete(0,"SLHLine3");
                                                                                                         ObjectDelete(0,"SLHLine4");
                                                                                                         ObjectDelete(0,"Text_SL_HLine1");
                                                                                                         ObjectDelete(0,"Text_TP_HLine1");
                                                                                                         ObjectDelete(0,"Text_SL_HLine2");
                                                                                                         ObjectDelete(0,"Text_TP_HLine2");

                                                                                                        }

                                                                                                      else
                                                                                                         if(sparam=="CADCHF")
                                                                                                           {
                                                                                                            ChartSetSymbolPeriod(ChartID(),"CADCHF",_Period);
                                                                                                            ObjectSetInteger(ChartID(),"CADCHF",OBJPROP_STATE,false);
                                                                                                            ObjectSetInteger(ChartID(),"CADCHF",OBJPROP_BGCOLOR,clrGreen);

                                                                                                            ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                            ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                            ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                            ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                            ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                            ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                            ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                            ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                            ObjectDelete(0,"SLHLine2");
                                                                                                            ObjectDelete(0,"SLHLine1");
                                                                                                            ObjectDelete(0,"TPHLine1");
                                                                                                            ObjectDelete(0,"TPHLine2");
                                                                                                            ObjectDelete(0,"SLHLine3");
                                                                                                            ObjectDelete(0,"SLHLine4");
                                                                                                            ObjectDelete(0,"Text_SL_HLine1");
                                                                                                            ObjectDelete(0,"Text_TP_HLine1");
                                                                                                            ObjectDelete(0,"Text_SL_HLine2");
                                                                                                            ObjectDelete(0,"Text_TP_HLine2");

                                                                                                           }


                                                                                                         else
                                                                                                            if(sparam=="EURAUD")
                                                                                                              {
                                                                                                               ChartSetSymbolPeriod(ChartID(),"EURAUD",_Period);
                                                                                                               ObjectSetInteger(ChartID(),"EURAUD",OBJPROP_STATE,false);
                                                                                                               ObjectSetInteger(ChartID(),"EURAUD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                               ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                               ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                               ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                               ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                               ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                               ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                               ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                               ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                               ObjectDelete(0,"SLHLine2");
                                                                                                               ObjectDelete(0,"SLHLine1");
                                                                                                               ObjectDelete(0,"TPHLine1");
                                                                                                               ObjectDelete(0,"TPHLine2");
                                                                                                               ObjectDelete(0,"SLHLine3");
                                                                                                               ObjectDelete(0,"SLHLine4");
                                                                                                               ObjectDelete(0,"Text_SL_HLine1");
                                                                                                               ObjectDelete(0,"Text_TP_HLine1");
                                                                                                               ObjectDelete(0,"Text_SL_HLine2");
                                                                                                               ObjectDelete(0,"Text_TP_HLine2");

                                                                                                              }


                                                                                                            else
                                                                                                               if(sparam=="EURCAD")
                                                                                                                 {
                                                                                                                  ChartSetSymbolPeriod(ChartID(),"EURCAD",_Period);
                                                                                                                  ObjectSetInteger(ChartID(),"EURCAD",OBJPROP_STATE,false);
                                                                                                                  ObjectSetInteger(ChartID(),"EURCAD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                  ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                  ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                  ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                  ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                  ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                  ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                  ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                  ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                  ObjectDelete(0,"SLHLine2");
                                                                                                                  ObjectDelete(0,"SLHLine1");
                                                                                                                  ObjectDelete(0,"TPHLine1");
                                                                                                                  ObjectDelete(0,"TPHLine2");
                                                                                                                  ObjectDelete(0,"SLHLine3");
                                                                                                                  ObjectDelete(0,"SLHLine4");
                                                                                                                  ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                  ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                  ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                  ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                 }


                                                                                                               else
                                                                                                                  if(sparam=="EURCHF")
                                                                                                                    {
                                                                                                                     ChartSetSymbolPeriod(ChartID(),"EURCHF",_Period);
                                                                                                                     ObjectSetInteger(ChartID(),"EURCHF",OBJPROP_STATE,false);
                                                                                                                     ObjectSetInteger(ChartID(),"EURCHF",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                     ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                     ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                     ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                     ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                     ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                     ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                     ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                     ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                     ObjectDelete(0,"SLHLine2");
                                                                                                                     ObjectDelete(0,"SLHLine1");
                                                                                                                     ObjectDelete(0,"TPHLine1");
                                                                                                                     ObjectDelete(0,"TPHLine2");
                                                                                                                     ObjectDelete(0,"SLHLine3");
                                                                                                                     ObjectDelete(0,"SLHLine4");
                                                                                                                     ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                     ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                     ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                     ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                    }

                                                                                                                  else
                                                                                                                     if(sparam=="EURGBP")
                                                                                                                       {
                                                                                                                        ChartSetSymbolPeriod(ChartID(),"EURGBP",_Period);
                                                                                                                        ObjectSetInteger(ChartID(),"EURGBP",OBJPROP_STATE,false);
                                                                                                                        ObjectSetInteger(ChartID(),"EURGBP",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                        ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                        ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                        ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                        ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                        ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                        ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                        ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                        ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                        ObjectDelete(0,"SLHLine2");
                                                                                                                        ObjectDelete(0,"SLHLine1");
                                                                                                                        ObjectDelete(0,"TPHLine1");
                                                                                                                        ObjectDelete(0,"TPHLine2");
                                                                                                                        ObjectDelete(0,"SLHLine3");
                                                                                                                        ObjectDelete(0,"SLHLine4");
                                                                                                                        ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                        ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                        ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                        ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                       }
                                                                                                                     else
                                                                                                                        if(sparam=="EURJPY")
                                                                                                                          {
                                                                                                                           ChartSetSymbolPeriod(ChartID(),"EURJPY",_Period);
                                                                                                                           ObjectSetInteger(ChartID(),"EURJPY",OBJPROP_STATE,false);
                                                                                                                           ObjectSetInteger(ChartID(),"EURJPY",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                           ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                           ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                           ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                           ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                           ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                           ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                           ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                           ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                           ObjectDelete(0,"SLHLine2");
                                                                                                                           ObjectDelete(0,"SLHLine1");
                                                                                                                           ObjectDelete(0,"TPHLine1");
                                                                                                                           ObjectDelete(0,"TPHLine2");
                                                                                                                           ObjectDelete(0,"SLHLine3");
                                                                                                                           ObjectDelete(0,"SLHLine4");
                                                                                                                           ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                           ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                           ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                           ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                          }
                                                                                                                        else
                                                                                                                           if(sparam=="EURNZD")
                                                                                                                             {
                                                                                                                              ChartSetSymbolPeriod(ChartID(),"EURNZD",_Period);
                                                                                                                              ObjectSetInteger(ChartID(),"EURNZD",OBJPROP_STATE,false);
                                                                                                                              ObjectSetInteger(ChartID(),"EURNZD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                              ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                              ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                              ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                              ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                              ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                              ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                              ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                              ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                              ObjectDelete(0,"SLHLine2");
                                                                                                                              ObjectDelete(0,"SLHLine1");
                                                                                                                              ObjectDelete(0,"TPHLine1");
                                                                                                                              ObjectDelete(0,"TPHLine2");
                                                                                                                              ObjectDelete(0,"SLHLine3");
                                                                                                                              ObjectDelete(0,"SLHLine4");
                                                                                                                              ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                              ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                              ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                              ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                             }
                                                                                                                           else
                                                                                                                              if(sparam=="EURUSD")
                                                                                                                                {
                                                                                                                                 ChartSetSymbolPeriod(ChartID(),"EURUSD",_Period);
                                                                                                                                 ObjectSetInteger(ChartID(),"EURUSD",OBJPROP_STATE,false);
                                                                                                                                 ObjectSetInteger(ChartID(),"EURUSD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                 ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                 ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                 ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                 ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                 ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                 ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                 ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                 ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                                 ObjectDelete(0,"SLHLine2");
                                                                                                                                 ObjectDelete(0,"SLHLine1");
                                                                                                                                 ObjectDelete(0,"TPHLine1");
                                                                                                                                 ObjectDelete(0,"TPHLine2");
                                                                                                                                 ObjectDelete(0,"SLHLine3");
                                                                                                                                 ObjectDelete(0,"SLHLine4");
                                                                                                                                 ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                 ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                 ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                 ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                }
                                                                                                                              else
                                                                                                                                 if(sparam=="GBPAUD")
                                                                                                                                   {
                                                                                                                                    ChartSetSymbolPeriod(ChartID(),"GBPAUD",_Period);
                                                                                                                                    ObjectSetInteger(ChartID(),"GBPAUD",OBJPROP_STATE,false);
                                                                                                                                    ObjectSetInteger(ChartID(),"GBPAUD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                    ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                    ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                    ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                    ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                    ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                    ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                    ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                    ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                    ObjectDelete(0,"SLHLine2");
                                                                                                                                    ObjectDelete(0,"SLHLine1");
                                                                                                                                    ObjectDelete(0,"TPHLine1");
                                                                                                                                    ObjectDelete(0,"TPHLine2");
                                                                                                                                    ObjectDelete(0,"SLHLine3");
                                                                                                                                    ObjectDelete(0,"SLHLine4");
                                                                                                                                    ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                    ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                    ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                    ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                   }
                                                                                                                                 else
                                                                                                                                    if(sparam=="GBPCAD")
                                                                                                                                      {
                                                                                                                                       ChartSetSymbolPeriod(ChartID(),"GBPCAD",_Period);
                                                                                                                                       ObjectSetInteger(ChartID(),"GBPCAD",OBJPROP_STATE,false);
                                                                                                                                       ObjectSetInteger(ChartID(),"GBPCAD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                       ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                       ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                       ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                       ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                       ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                       ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                       ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                       ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                       ObjectDelete(0,"SLHLine2");
                                                                                                                                       ObjectDelete(0,"SLHLine1");
                                                                                                                                       ObjectDelete(0,"TPHLine1");
                                                                                                                                       ObjectDelete(0,"TPHLine2");
                                                                                                                                       ObjectDelete(0,"SLHLine3");
                                                                                                                                       ObjectDelete(0,"SLHLine4");
                                                                                                                                       ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                       ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                       ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                       ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                      }
                                                                                                                                    else
                                                                                                                                       if(sparam=="GBPJPY")
                                                                                                                                         {
                                                                                                                                          ChartSetSymbolPeriod(ChartID(),"GBPJPY",_Period);
                                                                                                                                          ObjectSetInteger(ChartID(),"GBPJPY",OBJPROP_STATE,false);
                                                                                                                                          ObjectSetInteger(ChartID(),"GBPJPY",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                          ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                          ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                          ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                          ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                          ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                          ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                          ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                          ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                          ObjectDelete(0,"SLHLine2");
                                                                                                                                          ObjectDelete(0,"SLHLine1");
                                                                                                                                          ObjectDelete(0,"TPHLine1");
                                                                                                                                          ObjectDelete(0,"TPHLine2");
                                                                                                                                          ObjectDelete(0,"SLHLine3");
                                                                                                                                          ObjectDelete(0,"SLHLine4");
                                                                                                                                          ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                          ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                          ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                          ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                         }
                                                                                                                                       else
                                                                                                                                          if(sparam=="GBPNZD")
                                                                                                                                            {
                                                                                                                                             ChartSetSymbolPeriod(ChartID(),"GBPNZD",_Period);
                                                                                                                                             ObjectSetInteger(ChartID(),"GBPNZD",OBJPROP_STATE,false);
                                                                                                                                             ObjectSetInteger(ChartID(),"GBPNZD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                             ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                             ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                             ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                             ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                             ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                             ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                             ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                             ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                                             ObjectDelete(0,"SLHLine2");
                                                                                                                                             ObjectDelete(0,"SLHLine1");
                                                                                                                                             ObjectDelete(0,"TPHLine1");
                                                                                                                                             ObjectDelete(0,"TPHLine2");
                                                                                                                                             ObjectDelete(0,"SLHLine3");
                                                                                                                                             ObjectDelete(0,"SLHLine4");
                                                                                                                                             ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                             ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                             ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                             ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                            }
                                                                                                                                          else
                                                                                                                                             if(sparam=="GBPNZD")
                                                                                                                                               {
                                                                                                                                                ChartSetSymbolPeriod(ChartID(),"GBPNZD",_Period);
                                                                                                                                                ObjectSetInteger(ChartID(),"GBPNZD",OBJPROP_STATE,false);
                                                                                                                                                ObjectSetInteger(ChartID(),"GBPNZD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                                                ObjectDelete(0,"SLHLine2");
                                                                                                                                                ObjectDelete(0,"SLHLine1");
                                                                                                                                                ObjectDelete(0,"TPHLine1");
                                                                                                                                                ObjectDelete(0,"TPHLine2");
                                                                                                                                                ObjectDelete(0,"SLHLine3");
                                                                                                                                                ObjectDelete(0,"SLHLine4");
                                                                                                                                                ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                               }
                                                                                                                                             else
                                                                                                                                                if(sparam=="GBPCHF")
                                                                                                                                                  {
                                                                                                                                                   ChartSetSymbolPeriod(ChartID(),"GBPCHF",_Period);
                                                                                                                                                   ObjectSetInteger(ChartID(),"GBPCHF",OBJPROP_STATE,false);
                                                                                                                                                   ObjectSetInteger(ChartID(),"GBPCHF",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                   ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                   ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                   ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                   ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                   ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                   ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                   ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                   ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                                                   ObjectDelete(0,"SLHLine2");
                                                                                                                                                   ObjectDelete(0,"SLHLine1");
                                                                                                                                                   ObjectDelete(0,"TPHLine1");
                                                                                                                                                   ObjectDelete(0,"TPHLine2");
                                                                                                                                                   ObjectDelete(0,"SLHLine3");
                                                                                                                                                   ObjectDelete(0,"SLHLine4");
                                                                                                                                                   ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                   ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                   ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                   ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                  }
                                                                                                                                                else
                                                                                                                                                   if(sparam=="GBPUSD")
                                                                                                                                                     {
                                                                                                                                                      ChartSetSymbolPeriod(ChartID(),"GBPUSD",_Period);
                                                                                                                                                      ObjectSetInteger(ChartID(),"GBPUSD",OBJPROP_STATE,false);
                                                                                                                                                      ObjectSetInteger(ChartID(),"GBPUSD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                                                      ObjectDelete(0,"SLHLine2");
                                                                                                                                                      ObjectDelete(0,"SLHLine1");
                                                                                                                                                      ObjectDelete(0,"TPHLine1");
                                                                                                                                                      ObjectDelete(0,"TPHLine2");
                                                                                                                                                      ObjectDelete(0,"SLHLine3");
                                                                                                                                                      ObjectDelete(0,"SLHLine4");
                                                                                                                                                      ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                      ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                      ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                      ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                     }
                                                                                                                                                   else
                                                                                                                                                      if(sparam=="NZDCAD")
                                                                                                                                                        {
                                                                                                                                                         ChartSetSymbolPeriod(ChartID(),"NZDCAD",_Period);
                                                                                                                                                         ObjectSetInteger(ChartID(),"NZDCAD",OBJPROP_STATE,false);
                                                                                                                                                         ObjectSetInteger(ChartID(),"NZDCAD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                         ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                         ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                         ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                         ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                         ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                         ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                         ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                         ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                         ObjectDelete(0,"SLHLine2");
                                                                                                                                                         ObjectDelete(0,"SLHLine1");
                                                                                                                                                         ObjectDelete(0,"TPHLine1");
                                                                                                                                                         ObjectDelete(0,"TPHLine2");
                                                                                                                                                         ObjectDelete(0,"SLHLine3");
                                                                                                                                                         ObjectDelete(0,"SLHLine4");
                                                                                                                                                         ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                         ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                         ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                         ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                        }
                                                                                                                                                      else
                                                                                                                                                         if(sparam=="NZDJPY")
                                                                                                                                                           {
                                                                                                                                                            ChartSetSymbolPeriod(ChartID(),"NZDJPY",_Period);
                                                                                                                                                            ObjectSetInteger(ChartID(),"NZDJPY",OBJPROP_STATE,false);
                                                                                                                                                            ObjectSetInteger(ChartID(),"NZDJPY",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                            ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                            ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                            ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                            ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                            ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                            ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                            ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                            ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                            ObjectDelete(0,"SLHLine2");
                                                                                                                                                            ObjectDelete(0,"SLHLine1");
                                                                                                                                                            ObjectDelete(0,"TPHLine1");
                                                                                                                                                            ObjectDelete(0,"TPHLine2");
                                                                                                                                                            ObjectDelete(0,"SLHLine3");
                                                                                                                                                            ObjectDelete(0,"SLHLine4");
                                                                                                                                                            ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                            ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                            ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                            ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                           }


                                                                                                                                                         else
                                                                                                                                                            if(sparam=="NZDUSD")
                                                                                                                                                              {
                                                                                                                                                               ChartSetSymbolPeriod(ChartID(),"NZDUSD",_Period);
                                                                                                                                                               ObjectSetInteger(ChartID(),"NZDUSD",OBJPROP_STATE,false);
                                                                                                                                                               ObjectSetInteger(ChartID(),"NZDUSD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                               ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                               ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                               ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                               ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                               ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                               ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                               ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                               ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                               ObjectDelete(0,"SLHLine2");
                                                                                                                                                               ObjectDelete(0,"SLHLine1");
                                                                                                                                                               ObjectDelete(0,"TPHLine1");
                                                                                                                                                               ObjectDelete(0,"TPHLine2");
                                                                                                                                                               ObjectDelete(0,"SLHLine3");
                                                                                                                                                               ObjectDelete(0,"SLHLine4");
                                                                                                                                                               ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                               ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                               ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                               ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                              }


                                                                                                                                                            else
                                                                                                                                                               if(sparam=="NZDCHF")
                                                                                                                                                                 {
                                                                                                                                                                  ChartSetSymbolPeriod(ChartID(),"NZDCHF",_Period);
                                                                                                                                                                  ObjectSetInteger(ChartID(),"NZDCHF",OBJPROP_STATE,false);
                                                                                                                                                                  ObjectSetInteger(ChartID(),"NZDCHF",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                  ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                  ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                  ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                  ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                  ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                  ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                  ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                  ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);


                                                                                                                                                                  ObjectDelete(0,"SLHLine2");
                                                                                                                                                                  ObjectDelete(0,"SLHLine1");
                                                                                                                                                                  ObjectDelete(0,"TPHLine1");
                                                                                                                                                                  ObjectDelete(0,"TPHLine2");
                                                                                                                                                                  ObjectDelete(0,"SLHLine3");
                                                                                                                                                                  ObjectDelete(0,"SLHLine4");

                                                                                                                                                                  ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                  ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                  ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                  ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                 }


                                                                                                                                                               else
                                                                                                                                                                  if(sparam=="USDCAD")
                                                                                                                                                                    {
                                                                                                                                                                     ChartSetSymbolPeriod(ChartID(),"USDCAD",_Period);
                                                                                                                                                                     ObjectSetInteger(ChartID(),"USDCAD",OBJPROP_STATE,false);
                                                                                                                                                                     ObjectSetInteger(ChartID(),"USDCAD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                     ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                     ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                     ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                     ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                     ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                     ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                     ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                     ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);
                                                                                                                                                                     ObjectDelete(0,"SLHLine2");
                                                                                                                                                                     ObjectDelete(0,"SLHLine1");
                                                                                                                                                                     ObjectDelete(0,"TPHLine1");
                                                                                                                                                                     ObjectDelete(0,"TPHLine2");
                                                                                                                                                                     ObjectDelete(0,"SLHLine3");
                                                                                                                                                                     ObjectDelete(0,"SLHLine4");
                                                                                                                                                                     ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                     ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                     ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                     ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                    }


                                                                                                                                                                  else
                                                                                                                                                                     if(sparam=="USDCHF")
                                                                                                                                                                       {
                                                                                                                                                                        ChartSetSymbolPeriod(ChartID(),"USDCHF",_Period);
                                                                                                                                                                        ObjectSetInteger(ChartID(),"USDCHF",OBJPROP_STATE,false);
                                                                                                                                                                        ObjectSetInteger(ChartID(),"USDCHF",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                        ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                        ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                        ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                        ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                        ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                        ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                        ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                        ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                        ObjectDelete(0,"SLHLine2");
                                                                                                                                                                        ObjectDelete(0,"SLHLine1");
                                                                                                                                                                        ObjectDelete(0,"TPHLine1");
                                                                                                                                                                        ObjectDelete(0,"TPHLine2");
                                                                                                                                                                        ObjectDelete(0,"SLHLine3");
                                                                                                                                                                        ObjectDelete(0,"SLHLine4");
                                                                                                                                                                        ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                        ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                        ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                        ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                       }


                                                                                                                                                                     else
                                                                                                                                                                        if(sparam=="USDJPY")
                                                                                                                                                                          {
                                                                                                                                                                           ChartSetSymbolPeriod(ChartID(),"USDJPY",_Period);
                                                                                                                                                                           ObjectSetInteger(ChartID(),"USDJPY",OBJPROP_STATE,false);
                                                                                                                                                                           ObjectSetInteger(ChartID(),"USDJPY",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                           ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                           ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                           ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                           ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                           ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                           ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                           ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                           ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);



                                                                                                                                                                           ObjectDelete(0,"SLHLine2");
                                                                                                                                                                           ObjectDelete(0,"SLHLine1");
                                                                                                                                                                           ObjectDelete(0,"TPHLine1");
                                                                                                                                                                           ObjectDelete(0,"TPHLine2");
                                                                                                                                                                           ObjectDelete(0,"SLHLine3");
                                                                                                                                                                           ObjectDelete(0,"SLHLine4");
                                                                                                                                                                           ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                           ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                           ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                           ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                          }


                                                                                                                                                                        else
                                                                                                                                                                           if(sparam=="XAUUSD")
                                                                                                                                                                             {
                                                                                                                                                                              ChartSetSymbolPeriod(ChartID(),"XAUUSD",_Period);
                                                                                                                                                                              ObjectSetInteger(ChartID(),"XAUUSD",OBJPROP_STATE,false);
                                                                                                                                                                              ObjectSetInteger(ChartID(),"XAUUSD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                              ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                              ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                              ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                              ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                              ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                              ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                              ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                              ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                              ObjectDelete(0,"SLHLine2");
                                                                                                                                                                              ObjectDelete(0,"SLHLine1");
                                                                                                                                                                              ObjectDelete(0,"TPHLine1");
                                                                                                                                                                              ObjectDelete(0,"TPHLine2");
                                                                                                                                                                              ObjectDelete(0,"SLHLine3");
                                                                                                                                                                              ObjectDelete(0,"SLHLine4");

                                                                                                                                                                              ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                              ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                              ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                              ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                             }


                                                                                                                                                                           else
                                                                                                                                                                              if(sparam=="BTCUSD")
                                                                                                                                                                                {
                                                                                                                                                                                 ChartSetSymbolPeriod(ChartID(),"BTCUSD",_Period);
                                                                                                                                                                                 ObjectSetInteger(ChartID(),"BTCUSD",OBJPROP_STATE,false);
                                                                                                                                                                                 ObjectSetInteger(ChartID(),"BTCUSD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                                 ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                 ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                                 ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                 ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                 ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                 ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                                 ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                 ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);


                                                                                                                                                                                 ObjectDelete(0,"SLHLine2");
                                                                                                                                                                                 ObjectDelete(0,"SLHLine1");
                                                                                                                                                                                 ObjectDelete(0,"TPHLine1");
                                                                                                                                                                                 ObjectDelete(0,"TPHLine2");
                                                                                                                                                                                 ObjectDelete(0,"SLHLine3");
                                                                                                                                                                                 ObjectDelete(0,"SLHLine4");
                                                                                                                                                                                 ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                                 ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                                 ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                                 ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                                }


                                                                                                                                                                              else
                                                                                                                                                                                 if(sparam=="ETHUSD")
                                                                                                                                                                                   {
                                                                                                                                                                                    ChartSetSymbolPeriod(ChartID(),"ETHUSD",_Period);
                                                                                                                                                                                    ObjectSetInteger(ChartID(),"ETHUSD",OBJPROP_STATE,false);
                                                                                                                                                                                    ObjectSetInteger(ChartID(),"ETHUSD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                                    ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                    ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                                    ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                    ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                    ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                    ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                                    ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                    ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);


                                                                                                                                                                                    ObjectDelete(0,"SLHLine2");
                                                                                                                                                                                    ObjectDelete(0,"SLHLine1");
                                                                                                                                                                                    ObjectDelete(0,"TPHLine1");
                                                                                                                                                                                    ObjectDelete(0,"TPHLine2");
                                                                                                                                                                                    ObjectDelete(0,"SLHLine3");
                                                                                                                                                                                    ObjectDelete(0,"SLHLine4");
                                                                                                                                                                                    ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                                    ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                                    ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                                    ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                                   }
                                                                                                                                                                                 else
                                                                                                                                                                                    if(sparam=="XRPUSD")
                                                                                                                                                                                      {
                                                                                                                                                                                       ChartSetSymbolPeriod(ChartID(),"XRPUSD",_Period);
                                                                                                                                                                                       ObjectSetInteger(ChartID(),"XRPUSD",OBJPROP_STATE,false);
                                                                                                                                                                                       ObjectSetInteger(ChartID(),"XRPUSD",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                                       ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                       ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                                       ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                       ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                       ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                       ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                                       ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                       ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                       ObjectDelete(0,"SLHLine2");
                                                                                                                                                                                       ObjectDelete(0,"SLHLine1");
                                                                                                                                                                                       ObjectDelete(0,"TPHLine1");
                                                                                                                                                                                       ObjectDelete(0,"TPHLine2");
                                                                                                                                                                                       ObjectDelete(0,"SLHLine3");
                                                                                                                                                                                       ObjectDelete(0,"SLHLine4");
                                                                                                                                                                                       ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                                       ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                                       ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                                       ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                                      }

                                                                                                                                                                                    else
                                                                                                                                                                                       if(sparam=="US100")
                                                                                                                                                                                         {
                                                                                                                                                                                          ChartSetSymbolPeriod(ChartID(),"US100",_Period);
                                                                                                                                                                                          ObjectSetInteger(ChartID(),"US100",OBJPROP_STATE,false);
                                                                                                                                                                                          ObjectSetInteger(ChartID(),"US100",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                                          ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                          ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                                          ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                          ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                          ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                          ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                                          ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                          ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                          ObjectDelete(0,"SLHLine2");
                                                                                                                                                                                          ObjectDelete(0,"SLHLine1");
                                                                                                                                                                                          ObjectDelete(0,"TPHLine1");
                                                                                                                                                                                          ObjectDelete(0,"TPHLine2");
                                                                                                                                                                                          ObjectDelete(0,"SLHLine3");
                                                                                                                                                                                          ObjectDelete(0,"SLHLine4");
                                                                                                                                                                                          ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                                          ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                                          ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                                          ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                                         }



                                                                                                                                                                                       else
                                                                                                                                                                                          if(sparam=="US500")
                                                                                                                                                                                            {
                                                                                                                                                                                             ChartSetSymbolPeriod(ChartID(),"US500",_Period);
                                                                                                                                                                                             ObjectSetInteger(ChartID(),"US500",OBJPROP_STATE,false);
                                                                                                                                                                                             ObjectSetInteger(ChartID(),"US500",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                                             ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                             ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                                             ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                             ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                             ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                             ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                                             ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                             ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                             ObjectDelete(0,"SLHLine2");
                                                                                                                                                                                             ObjectDelete(0,"SLHLine1");
                                                                                                                                                                                             ObjectDelete(0,"TPHLine1");
                                                                                                                                                                                             ObjectDelete(0,"TPHLine2");
                                                                                                                                                                                             ObjectDelete(0,"SLHLine3");
                                                                                                                                                                                             ObjectDelete(0,"SLHLine4");
                                                                                                                                                                                             ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                                             ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                                             ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                                             ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                                            }


                                                                                                                                                                                          else
                                                                                                                                                                                             if(sparam=="US30")
                                                                                                                                                                                               {
                                                                                                                                                                                                ChartSetSymbolPeriod(ChartID(),"US30",_Period);
                                                                                                                                                                                                ObjectSetInteger(ChartID(),"US30",OBJPROP_STATE,false);
                                                                                                                                                                                                ObjectSetInteger(ChartID(),"US30",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                                                ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                                                ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                                ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                                                ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                                ObjectDelete(0,"SLHLine2");
                                                                                                                                                                                                ObjectDelete(0,"SLHLine1");
                                                                                                                                                                                                ObjectDelete(0,"TPHLine1");
                                                                                                                                                                                                ObjectDelete(0,"TPHLine2");
                                                                                                                                                                                                ObjectDelete(0,"SLHLine3");
                                                                                                                                                                                                ObjectDelete(0,"SLHLine4");
                                                                                                                                                                                                ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                                                ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                                                ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                                                ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                                               }


                                                                                                                                                                                             else
                                                                                                                                                                                                if(sparam=="HK50")
                                                                                                                                                                                                  {
                                                                                                                                                                                                   ChartSetSymbolPeriod(ChartID(),"HK50",_Period);
                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"HK50",OBJPROP_STATE,false);
                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"HK50",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                   ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);


                                                                                                                                                                                                   ObjectDelete(0,"SLHLine2");
                                                                                                                                                                                                   ObjectDelete(0,"SLHLine1");
                                                                                                                                                                                                   ObjectDelete(0,"TPHLine1");
                                                                                                                                                                                                   ObjectDelete(0,"TPHLine2");
                                                                                                                                                                                                   ObjectDelete(0,"SLHLine3");
                                                                                                                                                                                                   ObjectDelete(0,"SLHLine4");
                                                                                                                                                                                                   ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                                                   ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                                                   ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                                                   ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                                                  }




                                                                                                                                                                                                else
                                                                                                                                                                                                   if(sparam=="JP225")
                                                                                                                                                                                                     {
                                                                                                                                                                                                      ChartSetSymbolPeriod(ChartID(),"JP225",_Period);
                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"JP225",OBJPROP_STATE,false);
                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"JP225",OBJPROP_BGCOLOR,clrGreen);

                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
                                                                                                                                                                                                      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);


                                                                                                                                                                                                      ObjectDelete(0,"SLHLine2");
                                                                                                                                                                                                      ObjectDelete(0,"SLHLine1");
                                                                                                                                                                                                      ObjectDelete(0,"TPHLine1");
                                                                                                                                                                                                      ObjectDelete(0,"TPHLine2");
                                                                                                                                                                                                      ObjectDelete(0,"SLHLine3");
                                                                                                                                                                                                      ObjectDelete(0,"SLHLine4");
                                                                                                                                                                                                      ObjectDelete(0,"Text_SL_HLine1");
                                                                                                                                                                                                      ObjectDelete(0,"Text_TP_HLine1");
                                                                                                                                                                                                      ObjectDelete(0,"Text_SL_HLine2");
                                                                                                                                                                                                      ObjectDelete(0,"Text_TP_HLine2");

                                                                                                                                                                                                     }














                                                                                                                                                                                                   else
                                                                                                                                                                                                      if(sparam=="TradingTimeframeButton")
                                                                                                                                                                                                        {
                                                                                                                                                                                                         if(ObjectGetInteger(ChartID(),"TradingTimeframeButton",OBJPROP_STATE))
                                                                                                                                                                                                           {
                                                                                                                                                                                                            ChartSetSymbolPeriod(ChartID(),_Symbol,PERIOD_H4);
                                                                                                                                                                                                            ObjectSetString(ChartID(),"TradingTimeframeButton",OBJPROP_TEXT,"H4");


                                                                                                                                                                                                           }
                                                                                                                                                                                                         else
                                                                                                                                                                                                           {
                                                                                                                                                                                                            ChartSetSymbolPeriod(ChartID(),_Symbol,PERIOD_D1);
                                                                                                                                                                                                            ObjectSetString(ChartID(),"TradingTimeframeButton",OBJPROP_TEXT,"D1");

                                                                                                                                                                                                           }


                                                                                                                                                                                                        }




      if(sparam=="BuyButton"||sparam=="SellButton"||sparam=="BuyStop"||sparam=="SellStop")
        {

         if(!ObjectGetInteger(ChartID(),"LongOnly",OBJPROP_STATE))
            if(Lot_sizeTotrade-AllowedLotsize>0)
               MessageBox("Lot size is lager than allowed lot size","Lot Size Message",0);
         if(SymbolInfoInteger(_Symbol,SYMBOL_SPREAD)>StringToInteger(ObjectGetString(ChartID(),"SpreadMax",OBJPROP_TEXT)))
            MessageBox("Spread lager than allowed setting","Spread Message",0);
        }

     }



   if(ObjectGetInteger(ChartID(),"LongOnly",OBJPROP_STATE))
      Tradingpositontype="Long";
   else
      if(ObjectGetInteger(ChartID(),"ShortOnly",OBJPROP_STATE))
         Tradingpositontype="Short";
      else
         if(ObjectGetInteger(ChartID(),"Long&Short",OBJPROP_STATE))
            Tradingpositontype="Long&Short";
         else
            if(ObjectGetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE))

               Tradingpositontype="Buy_Stop";

            else
               if(ObjectGetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE))

                  Tradingpositontype="Sell_Stop";
               else
                  Tradingpositontype="";

  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+


void OnTick()

  {
//Protect Trading Account from over stopploss

   ScanSLtoProtectAccount();

// Not Allow to show "One_click on the chart"
   ChartSetInteger(ChartID(),CHART_SHOW_ONE_CLICK,0);

// Not allow to show chart in other Lower period timeframe

   if(_Symbol!="XAUUSD")
      if(_Period!=PERIOD_D1&&_Period!=PERIOD_H4&&_Period!=PERIOD_M1)
         ChartSetSymbolPeriod(ChartID(),_Symbol,TradingTimeframe);

   string checkTradingtimeframe=ObjectGetString(ChartID(),"TradingTimeframeButton",OBJPROP_TEXT);


   if(checkTradingtimeframe!=getstringoftimeframe(_Period))
     {

      ObjectSetString(ChartID(),"TradingTimeframeButton",OBJPROP_TEXT,getstringoftimeframe(_Period));

     }



///Show Type of Account
   string Account_type="DEMO ACCOUNT";
   if(AccountInfoInteger(ACCOUNT_TRADE_MODE)==ACCOUNT_TRADE_MODE_REAL)
      Account_type="REAL ACCOUNT";


   int SLPoint=StringToInteger(ObjectGetString(0,"SLValue",OBJPROP_TEXT));
   double RatioTPSL=StringToDouble(ObjectGetString(0,"P/LValue",OBJPROP_TEXT));


   double RiskAccount=StringToDouble(ObjectGetString(0,"%Risk",OBJPROP_TEXT));
   if(RiskAccount>100)
     {
      RiskAccount=100;
      ObjectSetString(0,"%Risk",OBJPROP_TEXT,"100");
     }



///RE-Calculate Losize/////////////////
   if(CalculateSLPoints(_Symbol,RiskAccount,SLPoint)==0)
     {
      ObjectSetString(ChartID(),"CalculateLotsButton",OBJPROP_TEXT,"0");

     }
   else
     {
      ObjectSetString(ChartID(),"CalculateLotsButton",OBJPROP_TEXT,(string)CalculateLotSize(_Symbol,RiskAccount,SLPoint));
      ObjectSetString(ChartID(),"CalculateSLButton",OBJPROP_TEXT,(string)CalculateSLPoints(_Symbol,RiskAccount,SLPoint));
     }

   double Recheck_Lot_sizeTotrade=StringToDouble(ObjectGetString(ChartID(),"TradeLotsizeEdit",OBJPROP_TEXT));
   double Recheck_CalculatedLot_Size=StringToDouble(ObjectGetString(ChartID(),"CalculateLotsButton",OBJPROP_TEXT));
   double Recheck_MaxLotsize=StringToDouble(ObjectGetString(ChartID(),"LotMax",OBJPROP_TEXT));
   Recheck_Lot_sizeTotrade=FilterLotSize(Recheck_Lot_sizeTotrade,Recheck_MaxLotsize,Recheck_CalculatedLot_Size);
   ObjectSetString(ChartID(),"TradeLotsizeEdit",OBJPROP_TEXT,(string)Recheck_Lot_sizeTotrade);



   if(Recheck_Lot_sizeTotrade<=0)
     {

      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
      ObjectSetInteger(ChartID(),"LongOnly",OBJPROP_STATE,false);

      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
      ObjectSetInteger(ChartID(),"BuyStopSelect",OBJPROP_STATE,false);

      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_BGCOLOR,clrWhiteSmoke);
      ObjectSetInteger(ChartID(),"ShortOnly",OBJPROP_STATE,false);

      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_BGCOLOR,clrWhiteSmoke);
      ObjectSetInteger(ChartID(),"SellStopSelect",OBJPROP_STATE,false);

      ObjectDelete(0,"SLHLine2");
      ObjectDelete(0,"SLHLine1");
      ObjectDelete(0,"TPHLine1");
      ObjectDelete(0,"TPHLine2");
      ObjectDelete(0,"SLHLine3");
      ObjectDelete(0,"SLHLine4");
     }



   if(TradingmodeSelection==0)
     {


      string   AllSymbolsString="";

      AllSymbolsString = "AUDCHF|AUDJPY|AUDNZD|AUDUSD|CADJPY|CADCHF|EURAUD|EURCAD|EURCHF|EURGBP|EURJPY|EURNZD|EURUSD|GBPAUD|GBPCAD|GBPJPY|GBPNZD|GBPCHF|GBPUSD|NZDCAD|NZDJPY|NZDUSD|NZDCHF|USDCAD|USDCHF|USDJPY|XAUUSD";

      int      NumberOfTradeableSymbols;
      string   SymbolArray[];

      if(TradeSymbol == "CURRENT")  //Override TradeSymbols input variable and use the current chart symbol only
        {
         NumberOfTradeableSymbols = 1;

         ArrayResize(SymbolArray, 1);
         SymbolArray[0] = Symbol();
         Print("EA will process ", SymbolArray[0], " only");
        }
      else
        {
         string TradeSymbolsToUse = "";

         if(TradeSymbol == "ALL")
            TradeSymbolsToUse = AllSymbolsString;
         else
            TradeSymbolsToUse = TradeSymbol;

         //CONVERT TradeSymbolsToUse TO THE STRING ARRAY SymbolArray
         NumberOfTradeableSymbols = StringSplit(TradeSymbolsToUse, '|', SymbolArray);
        }

      ///////////////////////////////Create an array of trading timeframes/////////////////////////////////////////////////////////



      for(int i=0; i<NumberOfTradeableSymbols; i++)
        {

         string symbol=SymbolArray[i];
         int Digitvalue=(int)SymbolInfoInteger(symbol,SYMBOL_DIGITS);
         double Point_Value=SymbolInfoDouble(symbol,SYMBOL_POINT);
         //long stopLevels = SymbolInfoInteger(symbol, SYMBOL_TRADE_STOPS_LEVEL)+SymbolInfoInteger(symbol, SYMBOL_SPREAD);
         double Bid=NormalizeDouble(SymbolInfoDouble(symbol,SYMBOL_BID),Digitvalue)
                    ,Ask=NormalizeDouble(SymbolInfoDouble(symbol,SYMBOL_ASK),Digitvalue);
         double Lot=CalculateLotSize(symbol,RiskAccount,SLPoint);
         //string Entrytrigger=EntryggerTest5(symbol,TradingTimeframe,TrendMovingAverageArray,PRICE_MEDIAN,100,20);

         string Entrytrigger=EntryggerTest6(symbol,TradingTimeframe,TrendMovingAverageArray,PRICE_MEDIAN,100,20,1,1);

         string RobortMagic="";
         RobortMagic=getstringoftimeframe(TradingTimeframe)+" - AUTO";
         double CurrentMarginLevel=AccountInfoDouble(ACCOUNT_MARGIN_LEVEL);


         if((CurrentMarginLevel>=MarginLevelLimit+50||PositionsTotal()==0)&&(CountNoOfPositionsOfCurrentSymbol(symbol,POSITION_TYPE_SELL,ORDER_TYPE_SELL)+CountNoOfPositionsOfCurrentSymbol(symbol,POSITION_TYPE_BUY,ORDER_TYPE_BUY))<MaxPositions
            &&SymbolInfoInteger(symbol,SYMBOL_SPREAD)<50)
            if(DynamicPositionNo(symbol,POSITION_TYPE_BUY))
               if(Entrytrigger=="Buy")
                 {
                  if(Tradingpositontype=="Long"||Tradingpositontype=="Long&Short")
                     trade.Buy(Lot,symbol,Ask,NormalizeDouble(Ask-Point_Value*CalculateSLPoints(symbol,RiskAccount,SLPoint),Digitvalue),NormalizeDouble(Ask+Point_Value*CalculateSLPoints(symbol,RiskAccount,SLPoint)*RatioTPSL,Digitvalue),RobortMagic);
                  //trade.Buy(Lotsize,symbol,Ask,NormalizeDouble(Ask-Point_Value*Autoplacestoploss,Digitvalue),0,PositioncommentTradingtimeframe);
                  //trade.Buy(Lotsize,symbol,Ask,0,0,PositioncommentTradingtimeframe);
                  //SendNotification(symbol+" BUY POSITION"+"-IN: "+getstringoftimeframe(SelectTradingTimeframe)+"-LOTSIZE: "+(string)Lotsize);
                  //Alert("Buy signal for currentcy pair: "+ symbol+"Timeframe: "+getstringoftimeframe(SelectTradingTimeframe));
                  //SendNotification("Buy signal for currentcy pair: "+symbol+"Timeframe: "+getstringoftimeframe(SelectTradingTimeframe));
                  //ChartOpen(symbol,SelectTradingTimeframe);

                  //Print(symbol,">>BUY SIGNAL<<","IN TIMEFRAME: ",(string)SelectTradingTimeframe);
                 }



         if((CurrentMarginLevel>=MarginLevelLimit+50||PositionsTotal()==0)&&(CountNoOfPositionsOfCurrentSymbol(symbol,POSITION_TYPE_SELL,ORDER_TYPE_SELL)+CountNoOfPositionsOfCurrentSymbol(symbol,POSITION_TYPE_BUY,ORDER_TYPE_BUY))<MaxPositions
            &&SymbolInfoInteger(symbol,SYMBOL_SPREAD)<50)
            if(DynamicPositionNo(symbol,POSITION_TYPE_SELL))
               if(Entrytrigger=="Sell")
                 {

                  if(Tradingpositontype=="Short"|| Tradingpositontype=="Long&Short")
                     trade.Sell(Lot,symbol,Bid,NormalizeDouble(Bid+Point_Value*CalculateSLPoints(symbol,RiskAccount,SLPoint),Digitvalue),NormalizeDouble(Bid-Point_Value*CalculateSLPoints(symbol,RiskAccount,SLPoint)*RatioTPSL,Digitvalue),RobortMagic);
                  //trade.Buy(Lotsize,symbol,Ask,NormalizeDouble(Ask-Point_Value*Autoplacestoploss,Digitvalue),0,PositioncommentTradingtimeframe);
                  //trade.Buy(Lotsize,symbol,Ask,0,0,PositioncommentTradingtimeframe);
                  //SendNotification(symbol+" BUY POSITION"+"-IN: "+getstringoftimeframe(SelectTradingTimeframe)+"-LOTSIZE: "+(string)Lotsize);
                  //Alert("Buy signal for currentcy pair: "+ symbol+"Timeframe: "+getstringoftimeframe(SelectTradingTimeframe));
                  //SendNotification("Buy signal for currentcy pair: "+symbol+"Timeframe: "+getstringoftimeframe(SelectTradingTimeframe));
                  //ChartOpen(symbol,SelectTradingTimeframe);
                  //Print(symbol,">>BUY SIGNAL<<","IN TIMEFRAME: ",(string)SelectTradingTimeframe);
                 }
        }
     }


////////////Trailing Stop function


   if(ObjectGetInteger(ChartID(),"NormalTrailingStop",OBJPROP_STATE))
     {
      trailingstopForcurrentsymbol(StringToInteger(ObjectGetString(ChartID(),"Activate_Point",OBJPROP_TEXT)),StringToInteger(ObjectGetString(ChartID(),"TSL",OBJPROP_TEXT)),"Normal");

     }

   else
      if(ObjectGetInteger(ChartID(),"FastTrailingStop",OBJPROP_STATE))
        {
         trailingstopForcurrentsymbol(StringToInteger(ObjectGetString(ChartID(),"Activate_Point",OBJPROP_TEXT)),StringToInteger(ObjectGetString(ChartID(),"TSL",OBJPROP_TEXT)),"Fast");

        }


   if(InitialBalance==0)
      InitialBalance=AccountInfoDouble(ACCOUNT_BALANCE);

   double DailyTarget=InitialBalance;



   CreateLabel(ChartID(),0,"Trading_SessionValue",3,1555-x,20-y,"SESSION: "+TradingSession(),15,clrBlack);
   CreateLabel(ChartID(),0,"GMT",3,1555-x,55-y,"GMT:   "+TimeToString(TimeGMT(),TIME_SECONDS),15,clrBlack);
   CreateLabel(ChartID(),0,"TimeDowncount",3,1555-x,80-y,UpdateTimeLeft(),12,clrDarkViolet);

   CreateLabel(ChartID(),0,"Initial_Balance",3,1855-x,110-y,"InitialBL",10,clrBlack);
   CreateLabel(ChartID(),0,"Initial_BalanceValue",3,1730-x,110-y,(string)InitialBalance,10,clrGreen);

   CreateLabel(ChartID(),0,"Account_Balance",3,1855-x,125-y,"Account Balance",10,clrBlack);
   CreateLabel(ChartID(),0,"Account_BalanceValue",3,1730-x,125-y,(string)NormalizeDouble(AccountInfoDouble(ACCOUNT_BALANCE),2),10,clrGreen);

   CreateLabel(ChartID(),0,"Equity",3,1855-x,140-y,"Equity",10,clrBlack);
   CreateLabel(ChartID(),0,"EquityValue",3,1730-x,140-y,(string)NormalizeDouble(AccountInfoDouble(ACCOUNT_EQUITY),2),10,clrGreen);

   CreateLabel(ChartID(),0,"FreeMargin",3,1855-x,155-y,"Free Margin",10,clrBlack);
   CreateLabel(ChartID(),0,"FreeMarginValue",3,1730-x,155-y,(string)NormalizeDouble(AccountInfoDouble(ACCOUNT_MARGIN_FREE),2),10,clrGreen);

   CreateLabel(ChartID(),0,"Margin_Level",3,1855-x,170-y,"Margin Level",10,clrBlack);
   CreateLabel(ChartID(),0,"Margin_LevelValue",3,1730-x,170-y,(string)int(AccountInfoDouble(ACCOUNT_MARGIN_LEVEL))+" /"+(string)(MarginLevelLimit+50),10,
               set_color(NormalizeDouble(AccountInfoDouble(ACCOUNT_MARGIN_LEVEL),2),MarginLevelLimit+50,clrGreen,clrRed,clrYellow));

   if((AccountInfoDouble(ACCOUNT_MARGIN_LEVEL)<=MarginLevelLimit+50)&&AccountInfoDouble(ACCOUNT_MARGIN_LEVEL)>150)
      MessageBox("Margin Level is low","Margin Level",0);
   else
      if(AccountInfoDouble(ACCOUNT_MARGIN_LEVEL)<150 && PositionsTotal()>0)
         MessageBox("Margin Level is very low","Margin Level",0);


   CreateLabel(ChartID(),0,"Profit",3,1855-x,185-y,"Profit",10,clrBlack);
   CreateLabel(ChartID(),0,"ProfitValue",3,1730-x,185-y,(string)NormalizeDouble(AccountInfoDouble(ACCOUNT_EQUITY)-InitialBalance,1)+" ("+(string)(int)NormalizeDouble(100*(AccountInfoDouble(ACCOUNT_EQUITY)-InitialBalance)/DailyTarget,0)+"%)",10,
               set_color(NormalizeDouble(AccountInfoDouble(ACCOUNT_EQUITY)-InitialBalance,2),0,clrGreen,clrRed,clrYellow));


   CreateLabel(ChartID(),"Total_Positions",3,1855-x,185-y,"Total Positions",10,clrBlack);
   CreateLabel(ChartID(),"Total_PositionsValue",3,1690-x,185-y,(string)PositionsTotal(),10,clrGreen);

   CreateLabel(ChartID(),0,"Total_Orders",3,1855-x,200-y,"Total Pending Orders",10,clrBlack);
   CreateLabel(ChartID(),0,"Total_OrdersValue",3,1690-x,200-y,(string)OrdersTotal(),10,clrGreen);
   CreateLabel(ChartID(),0,"Xspace1",3,1860-x,215-y,"-------------------------------------------------",13,clrGreen);


   CreateLabel(ChartID(),0,"TIMEFRAME",3,1855-x,230-y,"TimeFrame",10,clrBlack);
//CreateLabel(ChartID(),0,"TIMEFRAMEValue",3,1730-x,230-y,getstringoftimeframe(_Period),10,clrBlack);



   CreateLabel(ChartID(),0,"Symbol",3,1690-x,230-y,"Symbol ",10,clrBlack);
   CreateLabel(ChartID(),0,"SymbolText",3,1630-x,230-y,ChartSymbol(ChartID()),10,clrGreen);

   CreateLabel(ChartID(),0,"Spread",3,1690-x,250-y,"Spread ",10,clrBlack);
   CreateLabel(ChartID(),0,"SpreadText",3,1632-x,250-y,(string)SymbolInfoInteger(_Symbol,SYMBOL_SPREAD),10,set_color(50,SymbolInfoInteger(_Symbol,SYMBOL_SPREAD)));

   string SelectedTradingmodetext ="";
   if(TradingmodeSelection==0)
      SelectedTradingmodetext ="Auto";
   else
      CreateLabel(ChartID(),0,"RSI_INDICATOR",3,1855-x,290-y,"RSI",10,clrBlack);
   CreateLabel(ChartID(),0,"Xspace2",3,1860-x,310-y,"-------------------------------------------------",13,clrGreen);
   CreateLabel(ChartID(),0,"Position_management",3,1820-x,320-y,"Position Management",12,clrBlack);



   CreateLabel(ChartID(),0,"ShowProfitonPointtext1",3,1850-x,-40-y,_Symbol,16,clrViolet);
   CreateLabel(ChartID(),0,"Account_type",3,1850-x-100,-40-y,Account_type,16,clrRed);
   CreateLabel(ChartID(),0,"ShowProfitonPoint",3,1675-x-300,-40-y,string(showmoney())+"    USD",16,set_color(ShowProfit(),0,clrGreen,clrRed,clrBlack));

//To remove comments on current chart
   if(TimeGMT()-SetTimeCount>200)
      //ObjectSetString(ChartID(),"SLValue",OBJPROP_TEXT,"1000");
      if(AccountInfoDouble(ACCOUNT_EQUITY)-InitialBalance>=DailyTarget)
        {
         //CloseAllPosition();
         CreateLabel(ChartID(),"TargetCompletion",1000-x,-30-y,"Daily Target Is Completed",25,clrGreen);
         //PlaySound("::Files\\DailyTarget.wav");

        }

      else
         ObjectDelete(ChartID(),"TargetCompletion");
//if(AccountInfoDouble(ACCOUNT_MARGIN_LEVEL)<MarginLevelLimit&&PositionsTotal()!=0)
//PlaySound("::Files\\Margin Alert.wav");



   if(StringToInteger(ObjectGetString(ChartID(),"Activate_Point",OBJPROP_TEXT))<0)
      ObjectSetString(ChartID(),"Activate_Point",OBJPROP_TEXT,"100");

   if(StringToInteger(ObjectGetString(ChartID(),"TSL",OBJPROP_TEXT))<0)
      ObjectSetString(ChartID(),"TSL",OBJPROP_TEXT,"50");



   CreateLabel(0,0,"Moneytotrade",0,435,160,IntegerToString(NormalizeDouble(AccountInfoDouble(ACCOUNT_EQUITY)*RiskAccount/100,2))+" (USD)",12,clrBlack);
   CreateLabel(0,0,"Max_slLable",0,435,200, IntegerToString(CalculateMaxSLPoints(_Symbol,RiskAccount,SLPoint)),12,clrBlack);



////////////////////////////Show Variation Price
   MqlRates SymbolPrice[];
   ArraySetAsSeries(SymbolPrice,true);
   int SymbolPricevalue;
   SymbolPricevalue=CopyRates(_Symbol,PERIOD_D1,0,4,SymbolPrice);

   CreateLabel(0,0,"HighestPricevalue",0,400,350,IntegerToString((SymbolPrice[0].high-SymbolPrice[0].close)/_Point),10,clrBlack);
   CreateLabel(0,0,"LowestPricevalue",0,400,370,IntegerToString((SymbolPrice[0].low-SymbolPrice[0].close)/_Point),10,clrBlack);
   CreateLabel(0,0,"OpenPricevalue",0,400,390,IntegerToString((SymbolPrice[0].open-SymbolPrice[0].close)/_Point),10,clrBlack);



   string Tradingsignal= EntryggerTest5(_Symbol,TradingTimeframe,TrendMovingAverageArray,PRICE_MEDIAN,100,20);
   if(Tradingsignal=="Buy")
     {
      ObjectDelete(ChartID(),"Sell Signal");
      CreateLabel(0,0,"Buy Signal",0,1000,5,"Buy Signal",30,clrGreen);
     }
   else
      if(Tradingsignal=="Sell")
        {
         ObjectDelete(ChartID(),"Buy Signal");

         CreateLabel(0,0,"Sell Signal",0,1000,5,"Sell Signal",30,clrRed);
        }

      else
        {
         ObjectDelete(ChartID(),"Buy Signal");
         ObjectDelete(ChartID(),"Sell Signal");

        }


///////////////////////////////
   showtradedPosition(ObjectGetInteger(ChartID(),"ShowTradeLevels",OBJPROP_STATE));


//Save_Account(5, 450);


   if(ObjectFind(ChartID(), "SLHLine3") < 0 && ObjectFind(ChartID(), "SLHLine4") < 0)
     {
      Text_HLine(0, "Text_SL_HLine1", ObjectGetDouble(0, "SLHLine1", OBJPROP_PRICE), StringToDouble(ObjectGetString(ChartID(), "TradeLotsizeEdit", OBJPROP_TEXT)), "buy", "SL", "Current_Price");
      Text_HLine(0, "Text_TP_HLine1", ObjectGetDouble(0, "TPHLine1", OBJPROP_PRICE), StringToDouble(ObjectGetString(ChartID(), "TradeLotsizeEdit", OBJPROP_TEXT)), "buy", "TP", "Current_Price");

      Text_HLine(0, "Text_SL_HLine2", ObjectGetDouble(0, "SLHLine2", OBJPROP_PRICE), StringToDouble(ObjectGetString(ChartID(), "TradeLotsizeEdit", OBJPROP_TEXT)), "sell", "SL", "Current_Price");
      Text_HLine(0, "Text_TP_HLine2", ObjectGetDouble(0, "TPHLine2", OBJPROP_PRICE), StringToDouble(ObjectGetString(ChartID(), "TradeLotsizeEdit", OBJPROP_TEXT)), "sell", "TP", "Current_Price");

      if(ObjectFind(ChartID(), "SLHLine1") >= 0)
        {
         double minSLPrice1 = SymbolPrice[0].close - _Point * StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT));
         double maxSLPrice1 = SymbolPrice[0].close - _Point * CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint);

         double ModifiedSLPrice1=NormalizeDouble(SymbolPrice[0].close-_Point*(StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT))+ CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint))/2,_Digits);
         if(ObjectGetDouble(0, "SLHLine1", OBJPROP_PRICE) > NormalizeDouble(minSLPrice1, _Digits))
           {
            MessageBox("StopLoss is less than the minimum Stoploss (" + ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT) + ").\nPlease lower SLHLine1.", "Lower Stoploss Message");
            ObjectSetDouble(ChartID(), "SLHLine1", OBJPROP_PRICE, ModifiedSLPrice1);
           }
         else
            if(ObjectGetDouble(0, "SLHLine1", OBJPROP_PRICE) < NormalizeDouble(maxSLPrice1, _Digits))
              {
               MessageBox("StopLoss is over the maximum Stoploss allowed (" + CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint) + ").", "Over Stoploss Message");
               ObjectSetDouble(ChartID(), "SLHLine1", OBJPROP_PRICE, ModifiedSLPrice1);
              }
        }

      if(ObjectFind(ChartID(), "SLHLine2") >= 0)
        {
         double maxSLPrice2 = SymbolPrice[0].close + _Point * CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint);
         double minSLPrice2 = SymbolPrice[0].close + _Point * StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT));

         double ModifiedSLPrice2=NormalizeDouble(SymbolPrice[0].close+_Point*(StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT))+ CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint))/2,_Digits);



         if(ObjectGetDouble(0, "SLHLine2", OBJPROP_PRICE) < NormalizeDouble(minSLPrice2, _Digits))
           {
            MessageBox("StopLoss is less than the minimum Stoploss (" + ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT) + ").\nPlease raise SLHLine2.", "Lower Stoploss Message");
            ObjectSetDouble(ChartID(), "SLHLine2", OBJPROP_PRICE, ModifiedSLPrice2);
           }
         else
            if(ObjectGetDouble(0, "SLHLine2", OBJPROP_PRICE) > NormalizeDouble(maxSLPrice2, _Digits))
              {
               MessageBox("StopLoss is over the maximum SL allowed (" + CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint) + ").", "Over Stoploss Message");
               ObjectSetDouble(ChartID(), "SLHLine2", OBJPROP_PRICE, ModifiedSLPrice2);
              }
        }
     }


   else
     {
      Text_HLine(0, "Text_SL_HLine1", ObjectGetDouble(0, "SLHLine1", OBJPROP_PRICE), StringToDouble(ObjectGetString(ChartID(), "TradeLotsizeEdit", OBJPROP_TEXT)), "buy", "SL", "SLHLine34");
      Text_HLine(0, "Text_TP_HLine1", ObjectGetDouble(0, "TPHLine1", OBJPROP_PRICE), StringToDouble(ObjectGetString(ChartID(), "TradeLotsizeEdit", OBJPROP_TEXT)), "buy", "TP", "SLHLine34");

      if(ObjectFind(ChartID(), "SLHLine3") >= 0)
        {
         double ModifiedSLPrice3=NormalizeDouble(ObjectGetDouble(0, "SLHLine3", OBJPROP_PRICE) - _Point * ((StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT)) + CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint)) / 2), _Digits);

         if(ObjectGetDouble(0, "SLHLine3", OBJPROP_PRICE) - ObjectGetDouble(0, "SLHLine1", OBJPROP_PRICE) > NormalizeDouble(_Point * CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint), _Digits))
           {
            MessageBox("Stop loss is over the maximum stop loss allowed.\nPlease raise SLHLine1.", "Stoploss Message");
            ObjectSetDouble(0, "SLHLine1", OBJPROP_PRICE, ModifiedSLPrice3);
           }
         else
            if(ObjectGetDouble(0, "SLHLine3", OBJPROP_PRICE) - ObjectGetDouble(0, "SLHLine1", OBJPROP_PRICE) < NormalizeDouble(_Point * StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT)), _Digits))
              {
               MessageBox("Stop loss is lower than the minimum stop loss allowed.\nPlease lower down SLHLine1.", "Stoploss Message");
               ObjectSetDouble(0, "SLHLine1", OBJPROP_PRICE, ModifiedSLPrice3);
              }
        }

      Text_HLine(0, "Text_SL_HLine2", ObjectGetDouble(0, "SLHLine2", OBJPROP_PRICE), StringToDouble(ObjectGetString(ChartID(), "TradeLotsizeEdit", OBJPROP_TEXT)), "sell", "SL", "SLHLine34");
      Text_HLine(0, "Text_TP_HLine2", ObjectGetDouble(0, "TPHLine2", OBJPROP_PRICE), StringToDouble(ObjectGetString(ChartID(), "TradeLotsizeEdit", OBJPROP_TEXT)), "sell", "TP", "SLHLine34");

      if(ObjectFind(ChartID(), "SLHLine4") >= 0)
        {
         double maxSLPrice = SymbolPrice[0].close + _Point * StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT));
         if(ObjectGetDouble(0, "SLHLine2", OBJPROP_PRICE) - ObjectGetDouble(0, "SLHLine4", OBJPROP_PRICE) > NormalizeDouble(_Point * CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint), _Digits))
           {
            MessageBox("Stop loss is over the maximum stop loss allowed.\nPlease raise up SL line.", "Stoploss Message");
            ObjectSetDouble(ChartID(), "SLHLine2", OBJPROP_PRICE, NormalizeDouble(ObjectGetDouble(0, "SLHLine4", OBJPROP_PRICE) + _Point * (StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT)) + CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint)) / 2, _Digits));
           }
         else
            if(ObjectGetDouble(0, "SLHLine2", OBJPROP_PRICE) - ObjectGetDouble(0, "SLHLine4", OBJPROP_PRICE) < NormalizeDouble(_Point * StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT)), _Digits))
              {
               MessageBox("Stop loss is lower than the minimum stop loss allowed.\nPlease lower down SL line.", "Stoploss Message");
               ObjectSetDouble(ChartID(), "SLHLine2", OBJPROP_PRICE, NormalizeDouble(ObjectGetDouble(0, "SLHLine4", OBJPROP_PRICE) + _Point * (StringToDouble(ObjectGetString(ChartID(), "SLMin", OBJPROP_TEXT)) + CalculateMaxSLPoints(_Symbol, RiskAccount, SLPoint)) / 2, _Digits));
              }
        }
     }



//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
   if(ObjectFind(ChartID(), "SLHLine3") >= 0)
     {
      if(SymbolPrice[0].close >= ObjectGetDouble(ChartID(), "SLHLine3", OBJPROP_PRICE))
        {
         PlaySound("::Files\\buystop.wav");
         SendNotification("CẶP " + _Symbol + " GIÁ VƯỢT " + NormalizeDouble(ObjectGetDouble(ChartID(), "SLHLine3", OBJPROP_PRICE), 2));
         MessageBox("CẶP " + _Symbol + " GIÁ VƯỢT " + NormalizeDouble(ObjectGetDouble(ChartID(), "SLHLine3", OBJPROP_PRICE), 2), "Alert Price");
        }
     }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
   if(ObjectFind(ChartID(), "SLHLine4") >= 0)
     {
      if(SymbolPrice[0].close <= ObjectGetDouble(ChartID(), "SLHLine4", OBJPROP_PRICE))
        {
         PlaySound("::Files\\sellstop.wav");
         SendNotification("CẶP " + _Symbol + " GIÁ THẤP HƠN " + NormalizeDouble(ObjectGetDouble(ChartID(), "SLHLine4", OBJPROP_PRICE), 2));
         MessageBox("CẶP " + _Symbol + " GIÁ THẤP HƠN " + NormalizeDouble(ObjectGetDouble(ChartID(), "SLHLine4", OBJPROP_PRICE), 2), "Alert Price");
        }
     }


  }
//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void CreateSymbol(string symbol="EURUSD",int distancex=100,int sizex=100, int distancey=100,int sizey=50,color colorSymbol=clrWhite)
  {

   ObjectCreate
   (
      ChartID(),             //current chart
      symbol,         //object name
      OBJ_BUTTON,          //object type
      0,                   //in main window
      0,                   //No datetime
      0                    //No price

   );


   ObjectSetInteger(ChartID(),symbol,OBJPROP_BGCOLOR,colorSymbol);
   ObjectSetInteger(ChartID(),symbol,OBJPROP_XDISTANCE,distancex);
   ObjectSetInteger(ChartID(),symbol,OBJPROP_XSIZE,sizex);
   ObjectSetInteger(ChartID(),symbol,OBJPROP_YDISTANCE,distancey);
   ObjectSetInteger(ChartID(),symbol,OBJPROP_YSIZE,sizey);
   ObjectSetInteger(ChartID(),symbol,OBJPROP_CORNER,3);
   ObjectSetInteger(ChartID(),symbol,OBJPROP_FONTSIZE,6);
   ObjectSetString(ChartID(),symbol,OBJPROP_TEXT,symbol);
   ObjectSetInteger(ChartID(),symbol,OBJPROP_COLOR,clrBlack);
   ObjectSetInteger(ChartID(),symbol,OBJPROP_ALIGN,ALIGN_CENTER);

  }


void CreateEdit(int chart_id=0, int sub_windows=0,string EditObjectname="name",int corner_no=3,int distancex=100,int sizex=100, int distancey=100,int sizey=50,string Text="50")

  {

   ObjectCreate
   (
      chart_id,          //current chart
      EditObjectname, //object name
      OBJ_EDIT,             //object type
      sub_windows,                   //in main window
      0,                   //No datetime
      0                    //No price

   );


   ObjectSetInteger(ChartID(),EditObjectname,OBJPROP_BGCOLOR,clrWhiteSmoke);
   ObjectSetInteger(ChartID(),EditObjectname,OBJPROP_XDISTANCE,distancex);
   ObjectSetInteger(ChartID(),EditObjectname,OBJPROP_XSIZE,sizex);
   ObjectSetInteger(ChartID(),EditObjectname,OBJPROP_YDISTANCE,distancey);
   ObjectSetInteger(ChartID(),EditObjectname,OBJPROP_YSIZE,sizey);
   ObjectSetInteger(ChartID(),EditObjectname,OBJPROP_CORNER,corner_no);
   ObjectSetString(ChartID(),EditObjectname,OBJPROP_TEXT,Text);
   ObjectSetInteger(ChartID(),EditObjectname,OBJPROP_COLOR,clrBlack);
//ObjectSetDouble(ChartID(),EditObjectname,OBJPROP_PRICE,LotSize);
   ObjectSetInteger(ChartID(),EditObjectname,OBJPROP_ALIGN,ALIGN_CENTER);


  }


//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void CreateButton(long Chart_id=0,int sub_windows=0,string Button_name="", ENUM_OBJECT Type_Button=OBJ_BUTTON,color color_button=clrWhite,
                  string Text_Button="",int corner_no=3, int distancex=100, int sizex=100, int distancey=100, int sizey=100)
  {

   ObjectCreate(
      Chart_id,
      Button_name,
      Type_Button,
      sub_windows,
      0,
      0
   );

   ObjectSetInteger(Chart_id,Button_name,OBJPROP_BGCOLOR,color_button);
   ObjectSetInteger(Chart_id,Button_name,OBJPROP_XDISTANCE,distancex);
   ObjectSetInteger(Chart_id,Button_name,OBJPROP_XSIZE,sizex);
   ObjectSetInteger(Chart_id,Button_name,OBJPROP_YDISTANCE,distancey);
   ObjectSetInteger(Chart_id,Button_name,OBJPROP_YSIZE,sizey);
   ObjectSetInteger(Chart_id,Button_name,OBJPROP_CORNER,corner_no);
   ObjectSetInteger(Chart_id,Button_name,OBJPROP_ALIGN,ALIGN_CENTER);
   ObjectSetString(Chart_id,Button_name,OBJPROP_TEXT,Text_Button);
   ObjectSetInteger(Chart_id,Button_name,OBJPROP_COLOR,clrBlack);

  }



//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void CreateLabel(ulong Chart_ID=0,int sub_windows=0,string Name_Label="",int corner_no=3,int distancex=100,int distancey=100,string Text_Label="",int Font_Size=10,color Text_color=clrWhite)
  {
   ObjectCreate(Chart_ID, Name_Label, OBJ_LABEL, sub_windows, 0, 0);

   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_XDISTANCE, distancex);
   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_YDISTANCE, distancey);
   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_CORNER, corner_no);
   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_ALIGN, ALIGN_CENTER);

   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_COLOR, Text_color);
   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_FONTSIZE, Font_Size);
   ObjectSetString(Chart_ID, Name_Label, OBJPROP_TEXT, Text_Label);
   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_BACK, false);
   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_HIDDEN, true);
   ObjectSetInteger(Chart_ID, Name_Label, OBJPROP_ZORDER, 0);

  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void CreateRectangleLabel(ulong Chart_ID=0,string Name_RectangleLabel="",int distancex=100,int sizex=100,int distancey=100,int sizey=100,color color_RectangleLabel=clrWhiteSmoke)
  {
   ObjectCreate(Chart_ID, Name_RectangleLabel, OBJ_RECTANGLE_LABEL, 0, 0, 0);

   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_XDISTANCE, distancex);
   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_YDISTANCE, distancey);
   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_XSIZE, sizex);
   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_YSIZE, sizey);
   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_BGCOLOR, color_RectangleLabel);
   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_CORNER, 3);
   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_BACK, false);
   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_HIDDEN, true);
   ObjectSetInteger(Chart_ID, Name_RectangleLabel, OBJPROP_ZORDER, 0);

  }




//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
color set_color(double Main_Value=100,double Compared_Value=0,color clr1=clrGreen,color clr2=clrRed,color clr3=clrDarkOrange)
  {

   color set_color=clrBlack;
   if(Main_Value>Compared_Value)
      set_color=clr1;
   else
      if(Main_Value<Compared_Value)
         set_color=clr2;
      else
         if(Main_Value==Compared_Value)
            set_color=clr3;

   return(set_color);
  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
color set_textcolor(string Text)
  {
   color Text_color=clrBlack;
   if(Text=="ON"||Text=="Manual"||Text=="LONG"||Text=="BUY_STOP")
      Text_color=clrGreen;
   else
      if(Text=="OFF"||Text=="Auto"||Text=="SHORT"||Text=="SELL_STOP")
         Text_color=clrRed;


   return(Text_color);



  }


/////////Count number of positions of current symbol////////////////
int CountNoOfPositionsOfCurrentSymbol(string symbol,ENUM_POSITION_TYPE PositionType=POSITION_TYPE_BUY,ENUM_ORDER_TYPE OrderType=ORDER_TYPE_BUY)
  {
   int a = 0, b = 0;

   for(int i = PositionsTotal() - 1; i >= 0; i--)
     {
      ulong positionTicket = PositionGetTicket(i);
      if(PositionGetString(POSITION_SYMBOL) == symbol)
        {
         if(PositionGetInteger(POSITION_TYPE) == PositionType)
            a++;
        }
     }

   for(int i = OrdersTotal() - 1; i >= 0; i--)
     {
      ulong orderTicket = OrderGetTicket(i);
      if(OrderGetString(ORDER_SYMBOL) == symbol)
        {
         if(OrderGetInteger(ORDER_TYPE) == OrderType)
            b++;
        }
     }

   return a + b;

  }



//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
color setButtonColor(string symbol)
  {
   color ButtonColor=clrWhite;
   for(int i=PositionsTotal()-1; i>=0; i--)
     {
      ulong PositionTicket=PositionGetTicket(i);
      if(symbol==PositionGetSymbol(i))
        {

         if(PositionGetDouble(POSITION_PROFIT)>=0)
            ButtonColor=clrGreen;
         else
            ButtonColor=clrRed;
        }

     }

   if(symbol==_Symbol)
      ButtonColor=clrDarkTurquoise;



   return(ButtonColor);

  }


//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
double ShowProfit()
  {
   double currentSymbolProfit = 0;

   for(int i = PositionsTotal() - 1; i >= 0; i--)
     {
      ulong positionTicket = PositionGetTicket(i);
      string symbol = PositionGetSymbol(i);
      double pointValue = SymbolInfoDouble(symbol, SYMBOL_POINT);

      if(symbol == _Symbol)
        {
         if(PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY)
            currentSymbolProfit = (PositionGetDouble(POSITION_PRICE_CURRENT) - PositionGetDouble(POSITION_PRICE_OPEN)) / pointValue;
         else
            currentSymbolProfit = (-PositionGetDouble(POSITION_PRICE_CURRENT) + PositionGetDouble(POSITION_PRICE_OPEN)) / pointValue;
         break;
        }
     }

   return (int)currentSymbolProfit;
  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
double showmoney()
  {

   double currentSymbolProfit = 0;

   for(int i = PositionsTotal() - 1; i >= 0; i--)
     {
      ulong positionTicket = PositionGetTicket(i);
      double positionProfit = PositionGetDouble(POSITION_PROFIT);
      currentSymbolProfit += positionProfit;
     }

   return currentSymbolProfit;

  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
string ShowPositionType()
  {
   string PositionType = "-----";

   for(int i = PositionsTotal() - 1; i >= 0; i--)
     {
      ulong PositionTicket = PositionGetTicket(i);
      string symbol = PositionGetSymbol(i);

      if(symbol == _Symbol)
        {
         if(PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY)
            PositionType = PositionGetString(POSITION_COMMENT) + " - LONG";
         else
            if(PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL)
               PositionType = PositionGetString(POSITION_COMMENT) + " - SHORT";
         break;
        }
     }

   if(PositionType == "-----")
     {
      for(int i = OrdersTotal() - 1; i >= 0; i--)
        {
         ulong OrderTicket = OrderGetTicket(i);
         string symbol = OrderGetString(ORDER_SYMBOL);

         if(symbol == _Symbol)
           {
            if(OrderGetInteger(ORDER_TYPE) == ORDER_TYPE_BUY_STOP)
               PositionType = "BUY_STOP";
            else
               if(OrderGetInteger(ORDER_TYPE) == ORDER_TYPE_SELL_STOP)
                  PositionType = "SELL_STOP";
            break;
           }
        }
     }

   return PositionType;
  }


//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void trailingstopForcurrentsymbol(int Activate_point=0,int trailingStopPoint=50,string TypeOfTrailingStop="Normal")
  {
   for(int i=PositionsTotal()-1; i>=0; i--)
     {
      ulong PositionTicket=PositionGetTicket(i);
      string symbol=PositionGetString(POSITION_SYMBOL);
      //if(_Symbol==symbol)

      if(_Symbol==symbol)
         if(TypeOfTrailingStop=="Fast")
           {
            Activate_point=100;
            trailingStopPoint=50;
           }

         else
            if(TypeOfTrailingStop=="Normal")
              {
               if(symbol=="AUDCAD")
                 {
                  trailingStopPoint=110;
                  if(Activate_point<10)
                     Activate_point=10;
                 }
               else
                  if(trailingStopPoint==0)
                    {
                     trailingStopPoint=10;
                    }

              }
            else
               if(TypeOfTrailingStop!="Normal")
                  trailingStopPoint=1000000;
        {
         double CurrentStopLoss=PositionGetDouble(POSITION_SL);
         double CurrentTakeprofit=PositionGetDouble(POSITION_TP);
         double PositionPriceOpen;
         double PositionPriceCurrent=PositionGetDouble(POSITION_PRICE_CURRENT);
         int Digitvalue=(int)SymbolInfoInteger(symbol,SYMBOL_DIGITS);
         double Point_Value=SymbolInfoDouble(symbol,SYMBOL_POINT);
         double Ask=NormalizeDouble(SymbolInfoDouble(symbol,SYMBOL_ASK),Digitvalue);
         double Bid=NormalizeDouble(SymbolInfoDouble(symbol,SYMBOL_BID),Digitvalue);


         if(PositionGetInteger(POSITION_TYPE)==POSITION_TYPE_BUY)
           {

            if(symbol!="AUDCAD")
               if(PositionPriceCurrent>=NormalizeDouble(PositionGetDouble(POSITION_PRICE_OPEN)+Point_Value*200,Digitvalue))
                  if(CurrentStopLoss<=PositionGetDouble(POSITION_PRICE_OPEN))
                     trade.PositionModify(PositionTicket,NormalizeDouble(PositionGetDouble(POSITION_PRICE_OPEN)+20*Point_Value,Digitvalue),CurrentTakeprofit);



            if(Activate_point==0)
               PositionPriceOpen=PositionGetDouble(POSITION_PRICE_OPEN);
            else
               PositionPriceOpen=NormalizeDouble(PositionGetDouble(POSITION_PRICE_OPEN)+Activate_point*Point_Value,Digitvalue);


            if(PositionPriceCurrent>=NormalizeDouble(PositionPriceOpen+trailingStopPoint*Point_Value,Digitvalue))
               if(PositionPriceCurrent>=NormalizeDouble(CurrentStopLoss+trailingStopPoint*Point_Value,Digitvalue))
                 {
                  trade.PositionModify(PositionTicket,NormalizeDouble(PositionPriceCurrent-(trailingStopPoint-2)*Point_Value,Digitvalue),CurrentTakeprofit);
                 }
           }
         else
            if(PositionGetInteger(POSITION_TYPE)==POSITION_TYPE_SELL)
              {
               if(symbol!="AUDCAD")
                  if(PositionPriceCurrent<=NormalizeDouble(PositionGetDouble(POSITION_PRICE_OPEN)-Point_Value*200,Digitvalue))
                     if(CurrentStopLoss>=PositionGetDouble(POSITION_PRICE_OPEN))
                        trade.PositionModify(PositionTicket,NormalizeDouble(PositionGetDouble(POSITION_PRICE_OPEN)-20*Point_Value,Digitvalue),CurrentTakeprofit);


               if(Activate_point==0)
                  PositionPriceOpen=PositionGetDouble(POSITION_PRICE_OPEN);
               else
                  PositionPriceOpen=NormalizeDouble(PositionGetDouble(POSITION_PRICE_OPEN)-Activate_point*Point_Value,Digitvalue);


               if(PositionPriceCurrent<=NormalizeDouble(PositionPriceOpen-trailingStopPoint*Point_Value,Digitvalue))
                  if(CurrentStopLoss>=NormalizeDouble(PositionPriceCurrent+trailingStopPoint*Point_Value,Digitvalue)||CurrentStopLoss==0)
                    {

                     trade.PositionModify(PositionTicket,NormalizeDouble(PositionPriceCurrent+(trailingStopPoint-2)*Point_Value,Digitvalue),CurrentTakeprofit);
                    }

              }

        }

     }

  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
bool Checksymbolavaibility(string symbol)
  {
   for(int i = SymbolsTotal(true) - 1; i >= 0; i--)
     {
      if(symbol == SymbolName(i, true))
        {
         return true;
        }
     }
   return false;
  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
bool DynamicPositionNo(string symbol, ENUM_POSITION_TYPE typeofcomingPosition)
  {
   bool DynamicPositionNoValue = false;
   double CheckTPLatestCurrentPosition = 0;

   int a = 0, b = 0;

   for(int i = OrdersTotal() - 1; i >= 0; i--)
     {
      ulong GetOrderTicket = OrderGetTicket(i);
      string GetOrderSymbol = OrderGetString(ORDER_SYMBOL);
      if(symbol == GetOrderSymbol)
        {
         b = 1;
         break;
        }
     }

   if(b == 0)
     {
      for(int i = PositionsTotal() - 1; i >= 0; i--)
        {
         ulong GetPositionTicket = PositionGetTicket(i);
         string GetPositionSymbol = PositionGetSymbol(i);
         int PositionType = PositionGetInteger(POSITION_TYPE);
         CheckTPLatestCurrentPosition = PositionGetDouble(POSITION_PROFIT);

         if(symbol == GetPositionSymbol && PositionType == typeofcomingPosition)
           {
            a = 1;
            if(CheckTPLatestCurrentPosition > 0)
               DynamicPositionNoValue = true;
            else
               DynamicPositionNoValue = false;
            break;
           }
        }

      if(a == 0)
         DynamicPositionNoValue = true;
     }

   return DynamicPositionNoValue;
  }



//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
int IndicatorCheckAvailability(string NameOfIndicator = "", int Chart_ID = 0, int Sub_windows = 0)
  {
   int indicatorNo = -1; // Initialize with an invalid value

   int IndicatorTotal = ChartIndicatorsTotal(Chart_ID, Sub_windows);
   for(int i = 0; i < IndicatorTotal; i++)
     {
      string IndicatorName = ChartIndicatorName(Chart_ID, Sub_windows, i);

      if(IndicatorName == NameOfIndicator)
        {
         indicatorNo = i;
         break;
        }
     }

   return indicatorNo;
  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void showtradedPosition(bool show_trade_levels=true)
  {
   CreateLabel(ChartID(),0,"PositionInfo1",3,1855-x,400,"Symbol"+"          "+"Buy/Sell"+"        "+"Money"+"        "+"Point",11,clrBlack);
   CreateLabel(ChartID(),0,"PositionInfo2",3,1855-x,410,"-----------------------------------------------------------",11,clrBlack);
   for(int i=0; i<=150; i++)
     {
      ObjectDelete(ChartID(),i+"1");
      ObjectDelete(ChartID(),i+"2");
      ObjectDelete(ChartID(),i+"3");
      ObjectDelete(ChartID(),i+"4");
      ObjectDelete(ChartID(),"SLText"+i);
      ObjectDelete(ChartID(),"TPText"+i);
     }
   MqlRates SymbolPrice1[];
   ArraySetAsSeries(SymbolPrice1,true);
   int SymbolPricevalue1;
   SymbolPricevalue1=CopyRates(_Symbol,_Period,0,16,SymbolPrice1);
   for(int i=0; i<=PositionsTotal()-1; i++)
     {
      string typeofposition="";
      color Posi_clr=clrBlack;
      ulong PositionTicket=PositionGetTicket(i);
      string symbol=PositionGetString(POSITION_SYMBOL);
      int Digitvalue=(int)SymbolInfoInteger(symbol,SYMBOL_DIGITS);
      double Point_Value=SymbolInfoDouble(symbol,SYMBOL_POINT);
      double PositionSL=PositionGetDouble(POSITION_SL);
      double PositionTP=PositionGetDouble(POSITION_TP);
      double lotsize=PositionGetDouble(POSITION_VOLUME);
      int Profit=int (PositionGetDouble(POSITION_PROFIT));
      int ProfitinPoint=0;
      int SL_point;
      int TP_point;
      double TP_money;
      double SL_money;
      if(PositionSL==0)
         MessageBox("Vi the ticket number: "+PositionTicket+"  chua dat stoploss","StopLoss Message",0);
      if(PositionGetInteger(POSITION_TYPE)==POSITION_TYPE_BUY)
        {
         ProfitinPoint=(PositionGetDouble(POSITION_PRICE_CURRENT)-PositionGetDouble(POSITION_PRICE_OPEN))/Point_Value;
         typeofposition="Buy";
         Posi_clr=clrGreen;
         SL_point=(-PositionGetDouble(POSITION_PRICE_OPEN)+PositionSL)/Point_Value;
         SL_money=NormalizeDouble(CalculatePointValue(symbol)*SL_point*lotsize,2);
         TP_point=(PositionTP- PositionGetDouble(POSITION_PRICE_OPEN))/Point_Value;
         TP_money=NormalizeDouble(CalculatePointValue(symbol)*TP_point*lotsize,2);
        }
      else
        {
         ProfitinPoint=(-PositionGetDouble(POSITION_PRICE_CURRENT)+PositionGetDouble(POSITION_PRICE_OPEN))/Point_Value;
         typeofposition="Sell";
         Posi_clr=clrRed;
         SL_point=(PositionGetDouble(POSITION_PRICE_OPEN)-PositionSL)/Point_Value;
         SL_money=NormalizeDouble(CalculatePointValue(symbol)*SL_point*lotsize,2);
         TP_point=(-PositionTP+ PositionGetDouble(POSITION_PRICE_OPEN))/Point_Value;
         TP_money=NormalizeDouble(CalculatePointValue(symbol)*TP_point*lotsize,2);
        }
      CreateLabel(ChartID(),0,i+"1",3,1855-x,425+i*20,symbol,9,Posi_clr);
      CreateLabel(ChartID(),0,i+"2",3,1855-x-90,425+i*20,typeofposition,10,Posi_clr);
      CreateLabel(ChartID(),0,i+"3",3,1855-x-180,425+i*20,Profit,10,set_color(Profit,0,clrGreen,clrRed,clrBlack));
      CreateLabel(ChartID(),0,i+"4",3,1855-x-260,425+i*20,ProfitinPoint,10,set_color(ProfitinPoint,0,clrGreen,clrRed,clrBlack));
      SL_money=NormalizeDouble(SL_money,2);
      if(show_trade_levels)
        {
         ObjectCreate(
            ChartID(),
            "SLText"+i,
            OBJ_TEXT,
            0,
            SymbolPrice1[0].time,
            NormalizeDouble(PositionSL,_Digits)
         );
         ObjectCreate(
            ChartID(),
            "TPText"+i,
            OBJ_TEXT,
            0,
            SymbolPrice1[0].time,
            NormalizeDouble(PositionTP,_Digits)
         );
         ObjectSetInteger(ChartID(),"SLText"+i,OBJPROP_COLOR,clrRed);
         if(SL_money>0)
            ObjectSetInteger(ChartID(),"SLText"+i,OBJPROP_COLOR,clrLightGreen);
         ObjectSetString(ChartID(),"SLText"+i,OBJPROP_TEXT,"SL"+i+": "+SL_point+"  " +SL_money+" USD");
         ObjectSetInteger(ChartID(),"TPText"+i,OBJPROP_COLOR,clrLightGreen);
         ObjectSetString(ChartID(),"TPText"+i,OBJPROP_TEXT,"TP"+i+": "+TP_point+"  " +TP_money+" USD");
        }
     }
  }



//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
string EntryggerTest5(string symbol, ENUM_TIMEFRAMES Timeframe = PERIOD_M2, int MA_Period = 10, ENUM_APPLIED_PRICE TypeOfPrice = PRICE_MEDIAN, int MaxPoint = 150, int MinPoint = 30)
  {
   string EntryggerTest5 = "";

   MqlRates TradingTimeframePrice[];
   ArraySetAsSeries(TradingTimeframePrice, true);
   int TradingTimeframedata = CopyRates(symbol, Timeframe, 0, 4, TradingTimeframePrice);

   double MA_Array[];
   int MADefinition = iMA(symbol, Timeframe, MA_Period, 0, MODE_SMA, TypeOfPrice);
   ArraySetAsSeries(MA_Array, true);
   CopyBuffer(MADefinition, 0, 0, 4, MA_Array);

   double Trix_Array[];
   int TrixDefinition = iTriX(symbol, Timeframe, 16, PRICE_MEDIAN);
   ArraySetAsSeries(Trix_Array, true);
   CopyBuffer(TrixDefinition, 0, 0, 4, Trix_Array);

   double symPoint = SymbolInfoDouble(symbol, SYMBOL_POINT);
   int sym_digit = SymbolInfoInteger(symbol, SYMBOL_DIGITS);

// Buy signal
   if(MA_Array[0] > MA_Array[1] &&
      TradingTimeframePrice[0].close > NormalizeDouble(MA_Array[0] + 10 * symPoint, sym_digit) &&
      Trix_Array[0] > Trix_Array[1] && Trix_Array[1] > Trix_Array[2])
     {
      EntryggerTest5 = "Buy";
     }
// Sell signal
   else
      if(MA_Array[0] > MA_Array[1] &&
         TradingTimeframePrice[0].close < NormalizeDouble(MA_Array[0] - 10 * symPoint, sym_digit) &&
         Trix_Array[0] < Trix_Array[1] && Trix_Array[1] < Trix_Array[2])
        {
         EntryggerTest5 = "Sell";
        }

   return EntryggerTest5;
  }



//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void Save_Account(int Risk = 10, double InitialBalanceAccount = 500)
  {
   double Equity_Account = AccountInfoDouble(ACCOUNT_EQUITY);

   if(Equity_Account <= InitialBalanceAccount * (1 - Risk / 100))
     {
      int Alarm_Account = MessageBox("Equity is lower than " + Risk + "% of initial capital. Please close losing positions.",
                                     "Alert: Account Balance", MB_OKCANCEL);
     }
  }


//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void Rejectorders()
  {
   for(int i = OrdersTotal() - 1; i >= 0; i--)
     {
      long orderTicket = OrderGetTicket(i);
      trade.OrderDelete(orderTicket);
     }
  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
string EntryggerTest6(string symbol, ENUM_TIMEFRAMES Timeframe=PERIOD_M2,int MA_Period=10,ENUM_APPLIED_PRICE TypeOfPrice=PRICE_MEDIAN,int MaxPoint=150, int MinPoint=30,int before=2, int after=2)
  {
   string EntryggerTest6="";
   int TotalCandle=before+after;

   MqlRates TradingTimeframePrice[];
   ArraySetAsSeries(TradingTimeframePrice,true);
   int TradingTimeframedata;
   TradingTimeframedata=CopyRates(symbol,Timeframe,0,TotalCandle,TradingTimeframePrice);

   double MA_Array[];
   int MADefinition=iMA(symbol,Timeframe,MA_Period,0,MODE_SMA,TypeOfPrice);
   ArraySetAsSeries(MA_Array,true);
   CopyBuffer(MADefinition,0,0,TotalCandle,MA_Array);


   double Trix_Array[];
   int TrixDefinition=iTriX(symbol,Timeframe,16,PRICE_MEDIAN);
   ArraySetAsSeries(Trix_Array,true);
   CopyBuffer(TrixDefinition,0,0,TotalCandle,Trix_Array);



   double symPoint=SymbolInfoDouble(symbol,SYMBOL_POINT);
   int sym_digit=SymbolInfoInteger(symbol,SYMBOL_DIGITS);


//Buy signal


   if(MA_Array[0]>MA_Array[1])
      if(TradingTimeframePrice[0].close>NormalizeDouble(MA_Array[0]+10*symPoint,sym_digit))
         if(Trix_Array[0]>Trix_Array[1]&&Trix_Array[1]>Trix_Array[2])
            EntryggerTest6="Buy";

//Sell signal

   if(MA_Array[0]>MA_Array[1])
      if(TradingTimeframePrice[0].close<NormalizeDouble(MA_Array[0]-10*symPoint,sym_digit))
         if(Trix_Array[0]<Trix_Array[1]&&Trix_Array[1]<Trix_Array[2])
            EntryggerTest6="Sell";

////////////////////////////////////////////////


   string after_buysignal="buy";
   string after_sellsignal="sell";
   string before_buysignal="buy";
   string before_sellsignal="sell";


   for(int i=0; i<after;i++)

     {

      if(MA_Array[i]<MA_Array[i+1]
         ||(TradingTimeframePrice[i].close<NormalizeDouble(MA_Array[i]+10*symPoint,sym_digit))
         ||(Trix_Array[i]<Trix_Array[i+1]))
         after_buysignal="";
      break;

     }

   for(int i=after; i<TotalCandle;i++)

     {

      if(MA_Array[i]>MA_Array[i+1]
         //||(TradingTimeframePrice[i].close<NormalizeDouble(MA_Array[i]+10*symPoint,sym_digit))
         ||(Trix_Array[i]>Trix_Array[i+1]))
         before_buysignal="";
      break;

     }

   if(after_buysignal=="buy" && before_buysignal=="buy")

      EntryggerTest6="Buy";

   else
     {

      for(int i=0; i<after;i++)

        {

         if(MA_Array[i]>MA_Array[i+1]
            ||(TradingTimeframePrice[i].close>NormalizeDouble(MA_Array[i]+10*symPoint,sym_digit))
            ||(Trix_Array[i]>Trix_Array[i+1]))
            after_sellsignal="";
         break;
        }

      for(int i=after; i<TotalCandle;i++)

        {

         if(MA_Array[i]<MA_Array[i+1]
            //||(TradingTimeframePrice[i].close<NormalizeDouble(MA_Array[i]+10*symPoint,sym_digit))
            ||(Trix_Array[i]<Trix_Array[i+1]))
            before_sellsignal="";
         break;
        }

      if(after_sellsignal=="sell"&&before_sellsignal=="sell")

         EntryggerTest6="Sell";

     }

   return(EntryggerTest6);


  }


//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void HLineCreate(long Chart_id = 0, string HLine_name = "HLine", int SLpoint = 500, string TypeOfPosi = "buy")
  {
   MqlRates Priceinformation[];
   ArraySetAsSeries(Priceinformation, true);
   int Priceinformationvalue = CopyRates(_Symbol, _Period, 0, 3, Priceinformation);

   double Price = (TypeOfPosi == "buy") ? NormalizeDouble(Priceinformation[0].close - SLpoint * _Point, _Digits)
                  : NormalizeDouble(Priceinformation[0].close + SLpoint * _Point, _Digits);

   ObjectCreate(Chart_id, HLine_name, OBJ_HLINE, 0, Priceinformation[0].time, Price);
   ObjectSetInteger(0, HLine_name, OBJPROP_SELECTABLE, true);
   ObjectSetInteger(0, HLine_name, OBJPROP_SELECTED, true);
   ObjectSetInteger(0, HLine_name, OBJPROP_BACK, true);
  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void Text_HLine(long chart_id = 0, string Text_HLine = "", double Price = 2000, double Lotsize = 0.01, string TypeofPosi = "buy", string SL_TP = "SL", string Based_Price = "Current_Price")
  {
   MqlRates Priceinformation[];
   ArraySetAsSeries(Priceinformation, true);
   int Priceinformationvalue = CopyRates(_Symbol, _Period, 0, 3, Priceinformation);

   double SL_point;
   double SL_money;

   if(Based_Price == "Current_Price")
     {
      SL_point = (TypeofPosi == "buy") ? (Price - Priceinformation[0].close) / _Point : (-Price + Priceinformation[0].close) / _Point;
      SL_money = NormalizeDouble(Lotsize * SL_point * CalculatePointValue(_Symbol), 0);
     }
   else
     {
      double referencePrice = ObjectGetDouble(ChartID(), (TypeofPosi == "buy") ? "SLHLine3" : "SLHLine4", OBJPROP_PRICE);
      SL_point = (TypeofPosi == "buy") ? (Price - referencePrice) / _Point : (-Price + referencePrice) / _Point;
      SL_money = NormalizeDouble(Lotsize * SL_point * CalculatePointValue(_Symbol), 0);
     }
   SL_point=(int)SL_point;
   SL_money=(int)SL_money;


   ObjectCreate(chart_id, Text_HLine, OBJ_TEXT, 0, Priceinformation[0].time, Price);
   ObjectSetInteger(0, Text_HLine, OBJPROP_COLOR, clrWhite);
   ObjectSetString(0, Text_HLine, OBJPROP_TEXT, SL_TP + ": " + SL_point + " Point " + SL_money + " USD");
  }


//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
void ScanSLtoProtectAccount()
  {
   for(int i = PositionsTotal() - 1; i >= 0; i--)
     {
      ulong GetPositionTicket = PositionGetTicket(i);
      string GetPositionSymbol = PositionGetSymbol(i);
      int PositionType = PositionGetInteger(POSITION_TYPE);
      string Getpositioncomment = PositionGetString(POSITION_COMMENT);
      int SLMAX = StringToInteger(Getpositioncomment);

      double CurrentStopLoss = PositionGetDouble(POSITION_SL);
      double CurrentProfit = PositionGetDouble(POSITION_TP);
      double PositionPriceOpen = PositionGetDouble(POSITION_PRICE_OPEN);
      int Digitvalue = (int)SymbolInfoInteger(GetPositionSymbol, SYMBOL_DIGITS);
      double Point_Value = SymbolInfoDouble(GetPositionSymbol, SYMBOL_POINT);
      int stoplossPoint;

      if(PositionType == POSITION_TYPE_BUY)
        {
         stoplossPoint = (PositionPriceOpen - CurrentStopLoss) / Point_Value;
         if(stoplossPoint > SLMAX && SLMAX > 0)
           {
            int Messconfirm = MessageBox("Position No: " + GetPositionTicket +
                                         "   " + GetPositionSymbol + " CurrentStoploss is over SL_MAX. Please raise SL to save your account",
                                         "Current Stoploss is lower than SL_Max allowed", MB_OKCANCEL);
            if(Messconfirm)
              {
               trade.PositionModify(GetPositionTicket, NormalizeDouble(PositionPriceOpen - SLMAX * Point_Value, Digitvalue), CurrentProfit);
              }
           }
        }
      else
         if(PositionType == POSITION_TYPE_SELL)
           {
            stoplossPoint = (-PositionPriceOpen + CurrentStopLoss) / Point_Value;
            if(stoplossPoint > SLMAX && SLMAX > 0)
              {
               int Messconfirm = MessageBox("Position No: " + GetPositionTicket +
                                            "   " + GetPositionSymbol + " CurrentStoploss is over SL_MAX. Please lower SL to save your account",
                                            "Current Stoploss is over SL_Max allowed", MB_OKCANCEL);
               if(Messconfirm)
                 {
                  trade.PositionModify(GetPositionTicket, NormalizeDouble(PositionPriceOpen + SLMAX * Point_Value, Digitvalue), CurrentProfit);
                 }
              }
           }
     }
  }



//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
double FilterLotSize(double LotSizeToTrade = 2, double MaxLotsize = 5, double CalculateLotsize = 3)
  {
   double finalLotsize = 0;

   if(LotSizeToTrade >= MathMax(MaxLotsize, CalculateLotsize))
      finalLotsize = MathMin(MaxLotsize, CalculateLotsize);
   else
     {
      if(CalculateLotsize <= MaxLotsize)
         finalLotsize = MathMin(LotSizeToTrade, CalculateLotsize);
      else
         finalLotsize = MathMin(LotSizeToTrade, MaxLotsize);
     }

   return finalLotsize;
  }
//+------------------------------------------------------------------+

<!---
dienbk/dienbk is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
