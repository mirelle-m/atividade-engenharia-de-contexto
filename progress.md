## Objetivo

Fazer `test_discount_applied_after_coupon` passar sem quebrar os outros testes de `tests/test_pricing.py`. A assinatura `apply_discount(order, coupon)` em `pricing.py` não pode mudar, porque o checkout depende dela.

## Abordagem

O bug estava em `_percent_off`: ele calculava sobre `order.subtotal`, e não sobre o total parcial. Com `chain(FIXED5, PERCENT10)` num pedido de 100.00, o certo é `Decimal("85.50")`. O usuário decidiu usar `Decimal` com `ROUND_HALF_UP` e `quantize(Decimal("0.01"))`, convertendo com `Decimal(str(order.subtotal))`. O projeto roda em Python 3.11 (`pyproject.toml`).

## Passos concluídos

Em `pricing.py`, importei `Decimal` e `ROUND_HALF_UP`, fiz `total = Decimal(str(order.subtotal))` e `_percent_off` passou a usar o total parcial com `quantize`. Agora `test_discount_applied_after_coupon` passa e `pytest tests/test_pricing.py -q` dá 1 failed, 6 passed.

## Falha atual

Falha `test_fixed_coupon_never_negative` (`tests/test_pricing.py:58`): com subtotal `Decimal("3.00")` e `chain(FIXED5)`, sai `Decimal('-2.00')` em vez de `Decimal("0.00")`. O próximo passo é fazer `_fixed_off` receber o total parcial e aplicar `max(Decimal("0"), ...)`. Depois, escrever `test_stacked_coupons` com dois cupons percentuais em sequência e rodar os testes de novo.
