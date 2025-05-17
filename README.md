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

- entry_orderid = tsl.order_placement(tradingsymbol=orderbook[name]['options_name'], exchange='NFO', quantity=orderbook[name]['qty'], price=0, trigger_price=0, order_type='MARKET', transaction_type='BUY', trade_type='MIS' )

                    print(f" Order Placed: {entry_orderid}")

                    timestamp = dt.datetime.now().strftime('%Y-%m-%d %H:%M:%S')

                
                    #  Place SL and Target Orders
                    stop_loss_price = round(last_close * 0.90, 1)  # 10% SL
                    target_price = round(last_close * 1.20, 1)      # 20% Target

                    # Stop Loss Sell Order (SL-M)
                    sl_order_id = tsl.order_placement(tradingsymbol=ce_name, exchange='NFO', quantity=quantity, price=0, trigger_price=stop_loss_price, order_type='MARKET', transaction_type='SELL', trade_type='MIS' )
                    print(f" Stop Loss Order Placed at {stop_loss_price}: {sl_order_id}")

                    


                    # Target Sell Order (LIMIT)
                    tgt_order_id = tsl.order_placement(tradingsymbol=ce_name, exchange='NFO', quantity=quantity, price=target_price, trigger_price=0, order_type='LIMIT', transaction_type='SELL', trade_type='MIS' )
                    print(f" Target Order Placed at {target_price}: {tgt_order_id}")


                    summary_message =(f" {spot_symbol} TRADE DETAILS\n" f"• Entry Price: ₹{last_close:.1f}\n" f"• Order ID: {entry_orderid}\n" f"• Stop Loss: ₹{stop_loss_price}\n" f"• Target: ₹{target_price}\n" f"• Quantity: {quantity}\n" f"• Trade Type: MIS\n" f"• Time: {timestamp}" )
                    tsl.send_telegram_alert(message=summary_message, receiver_chat_id=receiver_chat_id, bot_token=bot_token)

                    active_trades[spot_symbol] = {'in_position': True, 'entry_price': last_close, 'stop_loss_price': stop_loss_price, 'target_price': target_price, 'entry_order_id': entry_orderid }


- Telegram alerts

- tsl.send_telegram_alert(f" SL HIT for {symbol} at ₹{price}", receiver_chat_id, bot_token)
                exit_status = "SL HIT"
            elif price >= trade['target_price']:
                tsl.send_telegram_alert(f" TARGET HIT for {symbol} at ₹{price}", receiver_chat_id, bot_token)
                exit_status = "TARGET HIT"

- Trade log to Excel

- def log_to_excel(trade_data):
    df_new = pd.DataFrame([trade_data])
    try:
        wb = xw.Book(excel_file)
        sht = wb.sheets[sheet_name]
        existing_data = sht.range('A1').expand().value
        df_existing = pd.DataFrame(existing_data[1:], columns=existing_data[0])
        df_updated = pd.concat([df_existing, df_new], ignore_index=True)
        sht.clear_contents()
        sht.range("A1").value = [df_updated.columns.tolist()] + df_updated.values.tolist()
    except Exception:
        df_new.to_excel(excel_file, sheet_name=sheet_name, index=False)

## Usage
```bash
python broker_trade_bot.py
