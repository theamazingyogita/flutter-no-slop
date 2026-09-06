# Feature flow document

Write `docs/features/<feature>.md` when you build or substantially change a
feature, and update it in the same change as the code.

When you build or substantially change a feature, write
`docs/features/<feature>.md` alongside it. This is what lets the next person —
or the next agent session — understand the feature without reading every file.

Keep it short and factual:

```markdown
# Checkout

## What it does
One paragraph.

## Flow
1. User taps Pay on CartPage
2. CheckoutRequested event dispatched to CheckoutBloc
3. Bloc calls OrderRepository.submit()
4. Success -> CheckoutSuccess state -> navigate to ReceiptPage
5. Failure -> CheckoutFailure state -> error banner, cart preserved

## Files
- lib/checkout/view/checkout_page.dart
- lib/checkout/bloc/checkout_bloc.dart
- lib/checkout/data/order_repository.dart

## States
CheckoutInitial, CheckoutInProgress, CheckoutSuccess, CheckoutFailure

## Edge cases
- Network drop mid-submit: cart preserved, retry offered
- Empty cart: Pay button disabled
```

Update the doc in the same change as the code. A stale flow doc is worse than
none.

