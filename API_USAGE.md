# Game Core SDK - API Usage Guide

This SDK provides complete API interaction with the AI Games Platform services including configuration management, A/B testing, user authentication, and more.

## Installation

```bash
npm install game-sdk
```

## Initialization

```typescript
import { coreSDK } from 'game-sdk';

await coreSDK.init({
    app: 'my-game',
    version: '1.0.20',
    baseUrl: 'https://configs.artintgames.com',      // Configs service URL (optional)
    authUrl: 'https://auth.artintgames.com',       // Auth service URL (optional)
    sentryDsn: 'your-sentry-dsn',           // optional
});
```

### Initialization Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `app` | string | No | Application name |
| `version` | string | No | Application version |
| `baseUrl` | string | No | Base API URL (must include /v1) |
| `authUrl` | string | No | Auth API URL (must include /v1) |
| `sentryDsn` | string | No | Sentry DSN for error tracking |

## API Clients

The SDK exposes the following API clients through the `coreSDK` instance:

### 1. Configs API (`coreSDK.configs`)

Initialize and fetch configurations:

```typescript
// Initialize configuration
const config = await coreSDK.configs.init({
  v: '1.0.0',
  scopes: ['gameplay', 'ui'],
  segment: 'premium'
});

// Fetch configuration updates
const updatedConfig = await coreSDK.configs.fetch({
  v: '1.0.0',
  scopes: ['gameplay']
});
```

### 2. KV API (`coreSDK.kv`)

Initialize and fetch configurations:

```typescript
// Fetch all key-value pairs
const kvList = await coreSDK.kv.list();

// Example response
// {
//   items: [
//     { key: "settings.volume", value: "80" },
//     { key: "progress.level", value: "5" }
//   ]
// }
```

## ShowAd

How to Integrate Ads via SDK (Mediator-based)

1.	Initialize the SDK

```typescript
await coreSDK.init({});
```

2. Provide Ads Configuration
   Set ads configuration in the SDK config store.
   The config must include:
   • enabled: true
   • placements with placement id and ad type
   • optional defaults (timeouts, preload behavior)
```typescript
coreSDK.__setConfigForTests('ads', {
    enabled: true,
    defaults: {
        preloadOnStart: false,
    },
    placements: {
        reward_after_level: {
            type: 'rewarded',
            rewardType: 'coins',
            rewardAmount: 50,
            web: {
                mediator: 'fake',
            },
        },
    },
});
```

3. Create an Ad Mediator
   Implement a mediator object that follows the SDK BaseAdMediator interface.
   The mediator is responsible for:
   •	loading ads
   •	showing ads
   •	reporting the result back to the SDK
```typescript
class FakeAdsMediator {
    async init() {}
    async load() {}

    show(type, placement, placementConfig, done) {
        done({
            status: 'REWARD_GRANTED',
            canReward: true,
            rewardType: placementConfig.rewardType,
            rewardAmount: placementConfig.rewardAmount,
        });
    }
}
```
4. Register the Mediator
   Register the mediator in the SDK mediator registry.
   The SDK creates mediators automatically based on config..
```typescript
coreSDK.ads.registerMediator('fake', () => new FakeAdsMediator());
```
5. Initialize Ads Subsystem

Initialize ads after:
•	SDK initialization
•	ads config is set
•	mediator is registered

```typescript
await coreSDK.initAds();
```
6. Show an Ad

Call showAd(type, placement) from the SDK.

The SDK will:
•	validate config
•	ensure preload / readiness
•	select mediator by environment + config
•	delegate rendering to the mediator
•	return the final ad result

```typescript
coreSDK.showAd('rewarded', 'reward_after_level', result => {
  console.log(result);
});
```
Example of test mediator ( with base UI)
```typescript
(function (w) {
    if (!w.coreSDK) return;

    class FakeAdsMediator {
        async init() {}
        async load() {}

        show(type, placement, placementConfig, done) {
            const rewardText = `${placementConfig.rewardAmount} ${placementConfig.rewardType}`;

            const root = document.createElement('div');
            root.setAttribute('data-fake-ad', '');

            Object.assign(root.style, {
                position: 'fixed',
                inset: 0,
                background: 'rgba(0,0,0,0.85)',
                display: 'flex',
                alignItems: 'center',
                justifyContent: 'center',
                zIndex: 999999,
                fontFamily: 'system-ui, -apple-system',
                color: '#fff',
            });

            root.innerHTML = `
              <div style="
                background: linear-gradient(180deg,#111,#1a1a1a);
                padding:24px;
                border-radius:16px;
                width:320px;
                text-align:center;
                box-shadow: 0 20px 40px rgba(0,0,0,.6);
              ">
                <div style="font-size:20px;margin-bottom:8px">🎬 Rewarded Ad</div>

                <div style="opacity:.7;margin-bottom:12px">
                  Watch the ad to earn:
                </div>

                <div style="
                  font-size:22px;
                  font-weight:600;
                  margin-bottom:16px;
                ">
                  🪙 ${rewardText}
                </div>

                <div id="fake-ad-timer" style="opacity:.6;margin-bottom:16px">
                  Watching ad…
                </div>

                <div style="display:flex;gap:12px;justify-content:center">
                  <button id="fake-ad-cancel" style="
                    padding:10px 14px;
                    border-radius:10px;
                    border:none;
                    background:#333;
                    color:#fff;
                    cursor:pointer;
                  ">
                    Close
                  </button>

                  <button id="fake-ad-finish" style="
                    padding:10px 14px;
                    border-radius:10px;
                    border:none;
                    background:#4CAF50;
                    color:#000;
                    font-weight:600;
                    cursor:pointer;
                  ">
                    Finish Ad
                  </button>
                </div>
              </div>
            `;

            document.body.appendChild(root);

            const timerEl = root.querySelector('#fake-ad-timer');
            let seconds = 3;

            const interval = setInterval(() => {
                seconds--;
                if (seconds <= 0) {
                    clearInterval(interval);
                    timerEl.textContent = 'You can finish the ad';
                } else {
                    timerEl.textContent = `Watching ad… ${seconds}s`;
                }
            }, 1000);

            root.querySelector('#fake-ad-cancel').onclick = () => {
                clearInterval(interval);
                root.remove();
                done({
                    status: 'CANCELLED',
                    canReward: false,
                });
            };

            root.querySelector('#fake-ad-finish').onclick = () => {
                clearInterval(interval);
                root.remove();
                done({
                    status: 'REWARD_GRANTED',
                    canReward: true,
                    rewardType: placementConfig.rewardType,
                    rewardAmount: placementConfig.rewardAmount,
                });
            };
        }
    }

    // Register mediator
    coreSDK.ads.registerMediator('fake', () => new FakeAdsMediator());

    // Ads config
    coreSDK.__setConfigForTests('ads', {
        enabled: true,
        placements: {
            reward_after_level: {
                type: 'rewarded',
                rewardType: 'coins',
                rewardAmount: 50,
                web: {
                    mediator: 'fake',
                },
            },
        },
    });

    coreSDK.initAds();

    console.log('[FakeAds] ready');
})(window);
```
7. Handle Result (Optional)
   You can handle ad results via:
   •	callback passed to showAd
   •	returned promise
   •	SDK events (reward granted, cancelled, failed)
```
{
  status: 'REWARD_GRANTED' | 'CANCELLED' | 'SHOW_FAILED' | 'TIMEOUT',
  canReward: boolean,
  rewardType?: string,
  rewardAmount?: number
}
```

## Browser Support

The SDK works in all modern browsers that support:
- ES6 Modules
- Fetch API
- LocalStorage
- Promises/async-await

For legacy browser support, use appropriate polyfills.

## UMD Build

For script tag inclusion:

```html
<script src="/game-sdk.umd.js"></script>
<script>
    const sdk = window.coreSDK;
    sdk.init({
        app: 'my-game',
        version: '1.0.0',
        baseUrl: 'http://localhost/v1'
    }).then(() => {
        console.log('SDK ready');
    });
</script>
```

---

**Last Updated**: December 2025
**Version**: 1.0.20
