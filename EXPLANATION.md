# EXPLANATION.md

## What was the bug?

The bug was in the `HttpClient.request()` method's conditional logic. When `oauth2Token` was set to a plain JavaScript object (like `{ accessToken: "stale", expiresAt: 0 }`), the code tried to access the `expired` property. However, plain objects don't have the `expired` getter method that's defined in the `OAuth2Token` class, causing a runtime error.

## Why did it happen?

The original condition was:
```typescript
if (!this.oauth2Token || !(this.oauth2Token instanceof OAuth2Token) || this.oauth2Token.expired)
```

The problem was that `this.oauth2Token.expired` was evaluated even when the token was a plain object. JavaScript's logical OR (`||`) operator evaluates left-to-right, but if the second condition `!(this.oauth2Token instanceof OAuth2Token)` was true (meaning it's a plain object), the code still tried to evaluate `this.oauth2Token.expired`, which doesn't exist on plain objects.

## Why does your fix solve it?

The fix restructures the condition to only check `expired` when we know the token is an `OAuth2Token` instance:
```typescript
if (!this.oauth2Token || !(this.oauth2Token instanceof OAuth2Token) || (this.oauth2Token instanceof OAuth2Token && this.oauth2Token.expired))
```

Now the `expired` property is only accessed when we've confirmed the token is an `OAuth2Token` instance, preventing the runtime error.

## One realistic case / edge case your tests still don't cover

A realistic edge case not covered is when `oauth2Token` is set to a non-object value like a string, number, or boolean (e.g., `oauth2Token = "invalid-string"`). The current code would still attempt to check `instanceof OAuth2Token` which would work safely, but more robust validation could handle these edge cases explicitly.