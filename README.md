# Simple binance futures

## Attention

- Only support **binance**
- Only support **isolated**
- Only **long**, Short is not supported
- Not support **increasing margin**
- Not support **adjusting leverage** when holding positions,This is a limitation of Binance
- Please use the **2021.07** version of FT. There is no guarantee that the previous or later versions will support.Don't forget `pip install -r requirements.txt` after updating FT

## Prepare
- New binance users only support 20x leverage, please confirm whether your account supports higher leverage!
Make sure your API is enabled and have Futures permissions
Open futures after created the API, the created API still does not support futres, you must create a new API
- When the script starts, it will initialize Leverage, PositionsideDual, Isolated, so your position needs to be empty
- Make sure that the following methods are not customized `custom_stake_amount` `custom_stoploss` `confirm_trade_entry` `bot_loop_start`

## Modify config.json
- Must set `order_types.stoploss_on_exchange` to `true`.
- Muset set `bid_strategy.use_order_book` to `true`.
- Muset set `ask_strategy.use_order_book` to `true`.
- `exchange.ccxt_config` add `"options": {"defaultType": "future"}`
- `exchange.ccxt_async_config` add `"options": {"defaultType": "future"}`


## Modify strategy
```
from pandas import DataFrame

from freqtrade.strategy import stoploss_from_open
from pathlib import Path
import sys

sys.path.append(str(Path(__file__).parent))
import importlib

interface_futures_binance = importlib.import_module("interface_futures_binance")
importlib.reload(interface_futures_binance)


class SimpleBinanceFutures(interface_futures_binance.IFutures):
    use_custom_stoploss = True

    # Futures config
    # New binance users only support 20x leverage, please confirm whether your account supports higher leverage!
    # Make sure your API is enabled and have Futures permissions
    # (open futures after created the API, the created API still does not support futres, you must create a new API)
    _leverage = 10

    def custom_stoploss(
        self, pair: str, trade: 'Trade', current_time: 'datetime', current_rate: float, current_profit: float, **kwargs
    ) -> float:
        """
        Must! Must! Must use custom_stoploss, and turn on stoploss_on_exchange

        FT spot does not handle the logic of forced liquidation.
        Therefore, stoploss_on_exchange must be used to manually liquidate the position before the forced liquidation
        """
        p = locals()
        del p["self"], p["__class__"]
        parent_stoploss = super().custom_stoploss(**p)

        return max(stoploss_from_open(parent_stoploss, current_profit), -1)
```

## Other
- If you want to change the strategy, it is recommended to **restart** instead of using **web reload**.