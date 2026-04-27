# SDK Feature Flags

## 2) Флаги, влияющие на SDK и интеграционную логику

| Flag | Где применяется | Что меняет | Примечание для интеграции |
|---|---|---|---|
| `CUSTODIAL_WALLET` | SDK frontend | Показывает/скрывает иконку кошелька, пункт меню и роуты `custodial-wallet`. | Если выключить, пользователь не видит вход в кастодиальный кошелек в SDK. |
| `SDK_V2` | SDK frontend | Включает новый SDK exchanger flow. | Если флага нет, используется старый exchanger. |
| `WEB_CLIENT_ORDER_V1` | SDK frontend | Принудительно включает старый exchanger. | Имеет приоритет как rollback switch для UI-потока обмена. |
| `ORDER_V2_EXECUTE` | Exchange backend (`QuoteFacade`, `OrderFacade`) | Переключает execution-часть на V2 (create quote/order, reject). | Основной backend switch для V2-исполнения. |
| `ORDER_V2_VIEW` | Exchange backend (`OrderFacade`) | Переключает чтение заказов (active/list/details/report) на V2. | Держать синхронно с `ORDER_V2_EXECUTE` для консистентности. |
| `ORDER_LIMIT_V2` | Exchange backend (`LimitFacade`, `CalculationListener`) | Переключает расчет лимитов на V2 pipeline. | Может работать независимо от `ORDER_V2_EXECUTE`. |
| `DEPOSIT_LIMIT_V2` | Exchange backend (`BalanceOpHandler`) | Переключает deposit limit checks на V2 calculation flow. | Влияет на проверку min/max лимитов по депозитам. |
| `AEDEX_TOKEN` | Exchange backend validators | Разрешает операции с AED/AEDEX активами. | Без флага такие операции блокируются в валидации. |
| `HIDE_STATUS_CRYPTO_CARD_BLOCK` | Frontend payments/cards | Скрывает блок метода `STATUSBANK` (crypto card block) в списке платежных методов. | Чисто UI-скрытие метода в SDK/web карточках. |
| `HIDE_ALFA_CRYPTO_CARD_BLOCK` | Frontend payments/cards | Скрывает блок метода `ALFA_CRYPTO_CARD` в списке платежных методов. | Чисто UI-скрытие метода в SDK/web карточках. |
| `TEST_CRYSTAL_EXCEPTION_ON_CHECK` | Exchange backend (`TestCrystalClient`) | В тестовом Crystal-клиенте выбрасывает `RuntimeException("Crystal test check exception")` при AML check. | Работает только при `crystal.productionEnabled=false`; использовать для тестирования fail-сценариев AML. |
