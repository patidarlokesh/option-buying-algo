A Python-based intraday options trading bot using EMA, RSI, Supertrend

## Features
- Automated signal detection

  # Spot LTP for strike selection
        spot_symbols = list(symbol_to_spot.values())
        spot_ltp_data = tsl.get_ltp_data(names=spot_symbols)

        for fut_symbol in watchlist:
            # Signal on Future chart
            data_5 = tsl.get_historical_data(tradingsymbol=fut_symbol, exchange='NFO', timeframe="5")
            data = pd.DataFrame(data_5)

            data['ema_5'] = ta.ema(data['close'], length=21)
            data['ema_10'] = ta.ema(data['close'], length=50)
            data['rsi'] = ta.rsi(data['close'], length=14)
            supertrend = ta.supertrend(data['high'], data['low'], data['close'], length=7, multiplier=3)
            data = pd.concat([data, supertrend], axis=1)

            # Bullish condition
            data['bullish'] = (data['ema_5'] > data['ema_10']) & (data['rsi'] > 40) & (data['SUPERT_7_3.0'] < data['close'])

            if data['bullish'].iloc[-1]:
                print(f" SIGNAL BULLISH on {fut_symbol}")
                alert_message = (f" SIGNAL BULLISH on {fut_symbol}")
                tsl.send_telegram_alert(message=alert_message, receiver_chat_id=receiver_chat_id, bot_token=bot_token)



- ITM options strike selection

-
                    ce_name, pe_name, ce_itm_strike, pe_itm_strike = tsl.ITM_Strike_Selection(Underlying=spot_symbol, Expiry=0, ITM_count=2)
                    data_itm = tsl.get_historical_data(tradingsymbol=ce_name, exchange='NFO', timeframe="5")
                    
                    lot_size                          = tsl.get_lot_size(tradingsymbol = ce_name)
                    ltp_data = tsl.get_ltp_data(names=ce_name)
                    
                    last_close = pd.DataFrame(data_itm)['close'].iloc[-1]
                    quantity = int(capital_per_trade // (last_close * lot_size)) * lot_size
                    
                    name = spot_symbol  # or any name you want as the key
                    orderbook[name] = {'options_name': ce_name, 'qty': quantity }
                    
- Target/SL order placement
- Telegram alerts
- Trade log to Excel

## Usage
```bash
python dhan_trade_bot.py
