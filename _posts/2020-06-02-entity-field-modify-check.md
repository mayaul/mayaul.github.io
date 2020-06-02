---
layout: post
title: "entity 의 변경사항 체크"
excerpt: "entity 의 변경사항을 체크하고, 변경된 내용 저장하기"
date: 2020-06-02 00:00:00 +0900
comments: true
published : true
tag:
- hibernate
- hibernate history
- hibernate entity modify check
---
### 들어 가면서
* 우리가 envers 를 이용하면, entity 의 변경이력을 관리를 손쉽게 할 수 있습니다.
* 하지만, 변경된 필드에 대해서만 별도로 이력을 저장을 하고 싶다면 envers 를 이용해서는 한계가 있습니다.
* envers 를 이용해서, audit table 에서 특정 필드의 변경사항만 조회를 할 수는 있지만, 
    - 이 조회를 API로 구현을 해야하고, 그 API 가 사용자에게 노출이 될 목적의 데이터라면 별도의 데이터를 쌓는것이 DB에 과하게 부하를 줄 수 있어 제외하였습니다.

### using Interceptor
*  Interceptor 추가
``` java
public class Interceptor extends EmptyInterceptor {
    @Override
    public boolean onFlushDirty(Object entity,
                                Serializable id,
                                Object[] currentState,
                                Object[] previousState,
                                String[] propertyNames,
                                Type[] types)
        throws CallbackException {

        if (entity instanceof Entity) {
            log.info("[Interceptor] Entity 변경 되었습니다. property: {}, prev: {}, current: {}", propertyNames, previousState, currentState);
        }

        return false;
    }
}
```
* interceptor 설정하기 
    - application.yaml (full package 경로를 포함한 class path)
    ``` java
    spring.jpa.properties.hibernate.session_factory.interceptor: com.github.mayaul.Interceptor
    ```

* 한계점
    - checking 해야하는 field 추가/변경/삭제 시에 관리비용 많이 들어갈 수 있다.
    - 뭐가 변경이 되었는지를 확인할려면, `propertyNames` 로 모든 field 를 조회를 해야한다.

### using Listener
* Listener 추가
```java
@Component
public class EntityUpdateEventListener implements PostUpdateEventListener {
    @Override
    public void onPostUpdate(PostUpdateEvent event) {
        if (event.getEntity() instanceof Entity) {

            log.info("[Interceptor] Entity 변경 되었습니다. event: {}", event);
        }
    }

    @Override
    public boolean requiresPostCommitHanding(EntityPersister persister) {
        return false;
    }
}
```
* Listener 등록
```java
@Component
public class HibernateListenerConfig {

    private final EntityUpdateEventListener entityUpdateEventListener;

    @PersistenceUnit
    private EntityManagerFactory emf;

    public HibernateListenerConfig(EntityUpdateEventListener entityUpdateEventListener) {
        this.entityUpdateEventListener = entityUpdateEventListener;
    }

    @PostConstruct
    protected void init() {
        SessionFactoryImpl sessionFactory = emf.unwrap(SessionFactoryImpl.class);
        EventListenerRegistry registry = sessionFactory.getServiceRegistry().getService(EventListenerRegistry.class);
        registry.getEventListenerGroup(EventType.POST_UPDATE).appendListener(entityUpdateEventListener);
    }
}
```

    - `PostUpdateEvent#getDirtyProperties` 에 변경된 필드가 있다.
        ```java
        event.getOldState()[event.dirtyProperties[0]]
        event.getState()[event.dirtyProperties[0]]
        event.getOldState()[event.dirtyProperties[1]]
        event.getState()[event.dirtyProperties[1]]
        ``` 
        이렇게 조회하면 변경된 필드에 대해서만 이전과 현재값을 알 수 있다.


### 참고 자료
> https://kwonnam.pe.kr/wiki/java/hibernate/interceptor
> https://stackoverflow.com/questions/43472893/hibernate-envers-get-only-changed-fields
> https://vladmihalcea.com/hibernate-event-listeners/
