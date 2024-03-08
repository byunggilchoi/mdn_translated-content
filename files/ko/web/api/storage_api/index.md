---
title: Storage API
slug: Web/API/Storage_API
page-type: web-api-overview
browser-compat: api.StorageManager
---

{{securecontext_header}}{{DefaultAPISidebar("Storage")}} {{AvailableInWorkers}}

[Storage 규격](https://storage.spec.whatwg.org) 문서에는 웹사이트가 사용자의 브라우저에 데이터를 저장하는 데 사용할 수 있는 모든 API 및 기술에 적용되는 공유 저장소 시스템의 정의를 다루고 있습니다.

Storage 규격에 따라 관리되는 웹사이트의 데이터에는 일반적으로 [IndexedDB 데이터베이스](/ko/docs/Web/API/IndexedDB_API) 및 [Cache API 데이터](/ko/docs/Web/API/Cache)가 포함되지만, [Web Storage API 데이터](/ko/docs/Web/API/Web_Storage_API)와 같은 다른 유형의 사이트 접근 가능한 데이터도 포함될 수 있습니다.

Storage API는 사용할 수 있는 저장 용량, 이미 사용된 저장 용량 뿐만 아니라 다른 것을 저장하려고 {{Glossary("사용자 에이전트")}}가 데이터를 삭제하려고 할 때 알림을 받을 지 여부를 조정하는 등의 기능을 웹사이트에게 제공합니다.

이 문서는 사용자 에이전트가 웹사이트 데이터를 저장하고 유지하는 방법에 대해서 다룹니다. 저장소 제한 및 제거에 대한 자세한 내용은 [브라우저 저장소 할당량과 그 제거 기준](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria) 문서를 참조하십시오.

이 문서에서는 사이트의 사용 가능한 저장소를 추정하는 데 사용되는 {{domxref("StorageManager")}} 인터페이스에 대해서도 다룹니다.

## 개념과 사용법

### 저장소 버킷

Storage 규격에 설명된 사이트의 데이터가 저장되는 저장소 시스템은 일반적으로 각 {{Glossary("origin")}}에 대해 단일 _버킷_으로 구성됩니다.

본질적으로 모든 웹사이트는 데이터를 두는 자체 저장 공간을 갖습니다. 그러나 경우에 따라 사용자 에이전트는 단일 origin의 데이터를 여러 다른 버킷에 저장하기로 결정할 수도 있습니다. 예를 들어, 이 origin이 다른 서드파티 origin에 포함된 경우입니다.

자세한 내용은 [브라우저가 다른 웹사이트의 데이터를 어떻게 분리합니까?](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria#브라우저가-다른-웹사이트의-데이터를-어떻게-분리합니까)를 참조하십시오.

### 버킷 모드

각 사이트 저장소 버킷에는 해당 버킷의 데이터 보유 정책을 나타내는 _모드_가 있습니다. 모드에는 아래의 두 종류가 있습니다:

- `"best-effort"`
  - : 사용자 에이전트는 버킷에 포함된 데이터를 최대한 오랫동안 유지하려고 합니다.하지만 저장 공간이 부족해지고 저장 공간 압력을 해소하기 위해 버킷을 비우울 필요가 생겨도 _사용자에게 경고하지 않습니다_.
- `"persistent"`
  - : 사용자 에이전트는 버킷에 포함된 데이터를 최대한 오랫동안 유지하려고 합니다.그리고 `"persistent"`가 적힌 버킷을 지우기 전에 `"best-effort"`인 버킷을 먼저 삭제합니다. "persistent" 버킷을 지워야 하는 경우 사용자 에이전트는 사용자에게 이를 알리고 필요에 따라 하나 이상의 "persistent" 버킷을 삭제합니다.

"persistent-storage" [사용자 권한](/ko/docs/Web/API/Permissions_API)을 필요로 하는 {{domxref("StorageManager.persist", "navigator.storage.persist()")}} 메서드를 사용해서 origin의 저장소 버킷 모드를 변경할 수 있습니다.

```js
if (navigator.storage && navigator.storage.persist) {
  navigator.storage.persist().then((persistent) => {
    if (persistent) {
      console.log("Storage will not be cleared except by explicit user action");
    } else {
      console.log("Storage may be cleared by the UA under storage pressure.");
    }
  });
}
```

{{domxref("StorageManager.persisted", "navigator.storage.persisted()")}} 메서드를 사용해서 origin의 저장소 버킷 모드가 persistent인지 여부를 확인할 수도 있습니다:

```js
if (navigator.storage && navigator.storage.persist) {
  navigator.storage.persisted().then((persistent) => {
    if (persistent) {
      console.log("Storage will not be cleared except by explicit user action");
    } else {
      console.log("Storage may be cleared by the UA under storage pressure.");
    }
  });
}
```

자세한 내용은 [브라우저에 저장된 데이터가 유지됩니까?](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria#브라우저에-저장된-데이터가-유지됩니까)를 참조하세요.

### 할당량과 사용량 추정치

사용자 에이전트는 어떤 매커니즘이든 간에 사용하여 해당 사이트에서 사용할 수 있는 최대 저장 용량을 결정합니다. 이 최대값이 origin의 **할당량**입니다. 사이트에서 사용 중인 공간의 양을 **사용량**이라고 합니다. 이 두 값은 모두 추정치입니다. 정확하지 않은 데에는 몇 가지 이유가 있습니다:

- 사용자 에이전트는 해당 값이 [핑거프린팅](/ko/docs/Glossary/Fingerprinting) 목적으로 사용되는 것을 방지하기 위해 특정 origin에서 사용되는 데이터의 정확한 크기를 모호하게 만듭니다.
- 저장된 데이터의 실제 물리적 크기를 줄이기 위해 중복 제거, 압축 및 기타 방법을 사용할 수도 있습니다.
- 할당량은 해당 origin에서 사용 가능한 공간을 보수적으로 추정한 것이며, 초과 사용을 방지하려면 기기의 사용 가능한 공간보다 작아야 합니다.

특정 origin의 추정된 할당량 및 사용량 값을 확인하려면 {{domxref("StorageManager.estimate", "navigator.storage.estimate()")}} 메서드를 사용하세요. 이 수치를 포함하는 객체를 반환할 수 있습니다. 예를 들어:

```js
navigator.storage.estimate().then((estimate) => {
  // estimate.quota is the estimated quota
  // estimate.usage is the estimated number of bytes used
});
```

origin이 저장할 수 있는 데이터의 양에 대한 자세한 내용은 [얼마나 많은 데이터를 저장할 수 있습니까?](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria#얼마나-많은-데이터를-저장할-수-있습니까)를 참조하세요.

### 데이터 제거

데이터 제거는 사용자 에이전트가 origin의 저장된 데이터를 삭제하는 절차입니다. 예를 들어, 데이터 저장 장치의 저장 공간이 부족할 때 이런 일이 발생할 수 있습니다.

origin이 저장한 데이터를 지울 때, origin의 버킷은 단일 개체로 처리됩니다. 이 origin이 저장한 전체 데이터가 삭제되는 것입니다.

버킷이 `"persistent"`로 설정된 경우, 그 데이터를 저장한 origin이나 사용자가 명시적으로 삭제하지 않는 한 사용자 에이전트에 의해 내용이 삭제되지 않습니다. 사용자가 "캐시 지우기" 또는 "최근 기록 지우기" 옵션을 선택하는 시나리오도 포함됩니다. 이 때는 사용자에게 persistent 사이트 저장소 버킷을 제거할 권한을 명시적으로 요청합니다.

자세한 내용은 [데이터는 언제 제거됩니까?](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria#데이터는-언제-제거됩니까)를 참조하세요.

## 인터페이스

- {{domxref("StorageManager")}}
  - : persistent 권한을 관리하고 사용 가능한 스토리지를 추정하기 위한 인터페이스를 제공합니다.

### 추가 인터페이스

- {{domxref("Navigator.storage")}} {{ReadOnlyInline}}
  - : persistent 권한을 관리하고 사이트별/앱별로 사용 가능한 저장소를 추정하는 데 사용되는 싱글톤 {{domxref("StorageManager")}} 객체를 반환합니다.
- {{domxref("WorkerNavigator.storage")}} {{ReadOnlyInline}}
  - : persistent 권한을 관리하고 사용 가능한 스토리지를 추정하기 위한 {{domxref("StorageManager")}} 인터페이스를 반환합니다.

## 명세서

{{Specifications}}

## 브라우저 호환성

{{Compat}}

## 같이 보기

- [Permissions API 사용하기](/ko/docs/Web/API/Permissions_API/Using_the_Permissions_API)
- [Storage for the web on web.dev](https://web.dev/articles/storage-for-the-web)
- [Persistent storage on web.dev](https://web.dev/articles/persistent-storage)
- [Chrome Web Storage and Quota Concepts](https://docs.google.com/document/d/19QemRTdIxYaJ4gkHYf2WWBNPbpuZQDNMpUVf8dQxj4U/edit)
