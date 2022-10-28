# @nixjs23n6/backoff-typescript

A small library which handles decaying constant/linear/exponential backOff

## Quick Setup

### ConstantBackoff

The ConstantBackoff will make the websocket wait a constant time between each connection retry. To use the Constant Backoff with a wait-time of 1 second:

```typescript
const backoff  = new ConstantBackoff(1000)
setTimeout(() => {
    // todo something
}, backoff)
```

### LinearBackoff

The LinearBackoff linearly increases the wait-time between connection-retries until an optional maximum is reached. To use the LinearBackoff to initially wait 0 seconds and increase the wait-time by 2 second with every retry until a maximum of 12 seconds is reached:

```typescript
const backoff  = new LinearBackoff(0, 2000, 12000)
setTimeout(() => {
    // todo something
}, backoff)
```

### ExponentialBackoff

The ExponentialBackoff doubles the backoff with every retry until a maximum is reached. This is modelled after the binary exponential-backoff algorithm used in computer-networking. To use the ExponentialBackoff that will produce the series [100, 200, 400, 800, 1600, 3200, 6400]:

```typescript
const backoff  = new ExponentialBackoff(100, 7)
setTimeout(() => {
    // todo something
}, backoff)
```

## Examples

### Redux-saga

```typescript
import { LinearBackOff, BaseBackOff } from '@nixjs23n6/backoff-typescript'

function* pollPriceSaga(backOff: BaseBackOff) {
    while (true) {
        try {
            const time = backOff.next()
            if (time === 15000) {
                backOff.reset()
            }

            const result = yield call(FETCH_PRICE)
            yield put(onFetchPriceSuccess(result))
            if (time > 0) yield delay(time)
            yield delay(time)
        } catch (err) {
            yield put(onFetchPriceFailure(err))
        }
    }
}

function* watchPriceSaga() {
    while (true) {
        yield take(coingeckoSlice.onStartFetchPrice)

        const backOff = new LinearBackOff(0, 10000, 15000)
        yield race({
            task: call(pollPriceSaga, backOff),
            cancel: take(coingeckoSlice.onStopFetchPrice)
        })
    }
}
```