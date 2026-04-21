# SFSY Loon Capture

## Subscription URL

```text
https://raw.githubusercontent.com/dengfhqqq/openclaw-daily/main/loon/sfsy_loon.conf
```

## Captured result

The script stores the final value in persistent store key:

```text
sfsy_cookie
```

Expected value format:

```text
sessionId=xxx;_login_mobile_=xxx;_login_user_id_=xxx
```

## Behavior

- Captures from host `mcs-mimp-web.sf-express.com`
- De-duplicates same value notifications for 120 seconds
- Keeps recapture available (no permanent lock)
