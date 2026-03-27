---
category: backend
created_at: '2026-03-28T00:30:19.962797'
id: 20260328003019
tags:
- spring-boot
- test
- firebase
- gcs
- profile
- kotlin
title: Spring Boot 테스트에서 외부 서비스 빈 격리 — @Profile("!test")
updated_at: '2026-03-28T00:30:19.962797'
---

## 문제

Firebase Admin SDK, GCS Storage 등 외부 서비스 빈은 `@PostConstruct`에서 실제 인증 정보를 요구한다.
테스트 환경에서 ADC(Application Default Credentials)가 없으면 컨텍스트 로딩 자체가 실패해 **모든 테스트가 터진다.**

```
Caused by: java.io.IOException: Your default credentials were not found.
    at com.runnershi.common.config.FirebaseConfig.initialize(FirebaseConfig.kt:33)
```

## 해결

외부 인증이 필요한 빈에 `@Profile("!test")` 추가:

```kotlin
@Configuration
@Profile("!test")
class FirebaseConfig { ... }

@Service
@Profile("!test")
class FcmService { ... }

@Configuration
@Profile("!test")
class GcsConfig { ... }
```

이 빈을 주입받는 서비스는 optional로 처리:

```kotlin
@Service
class NotificationService(
    private val fcmService: FcmService? = null  // 테스트 환경에서는 null
) {
    fun send(...) {
        if (fcmService != null && user.notificationEnabled) {
            fcmService.sendPush(...)
        }
    }
}
```

## 교훈

- 외부 API/인프라 빈은 설계 시부터 `@Profile("!test")` 고려
- 의존 서비스는 nullable optional 주입으로 테스트 안전성 확보
- `@ActiveProfiles("test")`가 선언된 테스트 base class 있으면 자동으로 적용됨