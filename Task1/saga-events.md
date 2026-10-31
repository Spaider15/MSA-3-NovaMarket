# Реестр событий саги оформления заказа

| Этап | Тип события | Название | Публикует | Подписчики |
|---|---|---|---|---|
| Покупатель подтвердил заказ, заказ создан | domain | OrderCreated | Order Service | Inventory Service, Cart Service |
| Товары зарезервированы на складе | domain | StockReserved | Inventory Service | Payment Service |
| Товара не хватает, резерв не создан | failure | StockReservationFailed | Inventory Service | Order Service |
| Платеж создан, покупатель переходит на платежную страницу | domain | PaymentInitiated | Payment Service | Order Service |
| Успешная оплата | domain | PaymentSucceeded | Payment Service | Order Service |
| Оплата отклонена или не выполнена вовремя | failure | PaymentFailed | Payment Service | Order Service |
| Заказ оплачен и готовится к отправке | domain | OrderPaid | Order Service | Inventory Service, Delivery Service, Notification Service |
| Заявка на доставку создана | domain | ShipmentCreated | Delivery Service | Order Service, Notification Service |
| Не удалось создать заявку на доставку | failure | ShipmentCreationFailed | Delivery Service | Order Service |
| Отмена заказа | compensation | OrderCancelled | Order Service | Inventory Service, Payment Service, Notification Service |
| Снятие резерва товаров | compensation | StockReleased | Inventory Service | Order Service |
| Возврат средств | compensation | RefundSucceeded | Payment Service | Order Service, Notification Service |

Как отрабатывают компенсации:

- `failure`-события читает только Order Service. Он переводит заказ в `CANCELLED` с причиной (`OUT_OF_STOCK`, `PAYMENT_FAILED`, `DELIVERY_FAILED`) и публикует `OrderCancelled`.
- Inventory Service по `OrderCancelled` снимает резерв, если он был, и публикует `StockReleased`.
- Payment Service по `OrderCancelled` возвращает деньги, если они уже списаны, и публикует `RefundSucceeded`. Если платеж еще не завершен, отменяет его в платежном шлюзе.
- Если компенсировать нечего (например, заказ отменен из-за нехватки товара и до оплаты не дошло), сервис просто фиксирует, что событие обработано.

Все обработчики идемпотентные: повторно пришедшее событие с тем же id ничего не меняет.
