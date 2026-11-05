# Feature flow document

Use this when the project already keeps feature docs, or the developer asks for
one. Follow the project's location if it has one, otherwise use
`docs/features/<feature>.md`. A good doc lets the next person, or the next agent
session, understand the feature without reading every file.

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

