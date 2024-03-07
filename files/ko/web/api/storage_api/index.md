---
title: Storage API
slug: Web/API/Storage_API
page-type: web-api-overview
browser-compat: api.StorageManager
---

{{securecontext_header}}{{DefaultAPISidebar("Storage")}} {{AvailableInWorkers}}

[Storage Standard](https://storage.spec.whatwg.org) 문서에는 웹사이트가 사용자의 브라우저에 데이터를 저장하는 데 사용할 수 있는 모든 API 및 기술에 적용되는 공유 저장소 시스템의 정의를 다루고 있습니다.

저장소 표준에 의해 관리되는 웹사이트의 데이터에는 일반적으로 [IndexedDB databases](/ko/docs/Web/API/IndexedDB_API) 및 [Cache API data](/ko/docs/Web/API/Cache)가 포함되지만, [Web Storage API data](/ko/docs/Web/API/Web_Storage_API)와 같은 다른 유형의 사이트 접근 가능한 데이터도 포함될 수 있습니다.

저장소 API는 웹사이트에게 사용할 수 있는 공간의 양, 이미 사용 중인 양 및 데이터를 확보하기 위해 다른 데이터를 만들기 위해 {{Glossary("user agent")}}가 데이터를 삭제하기 전에 알림을 받아야 하는지 여부를 제어하는 기능을 제공합니다.

The data stored for a website which is managed by the Storage Standard usually includes [IndexedDB databases](/ko/docs/Web/API/IndexedDB_API) and [Cache API data](/ko/docs/Web/API/Cache), but may include other kind of site-accessible data such as [Web Storage API data](/ko/docs/Web/API/Web_Storage_API).

The Storage API gives websites the ability to find out how much space they can use, how much they are already using, and even control whether or not they need to be alerted before the {{Glossary("user agent")}} disposes of data in order to make room for other things.

이 문서는 사용자 에이전트가 웹사이트 데이터를 저장하고 유지하는 방법에 대한 개요를 제공합니다. 저장소 제한 및 제거에 대한 자세한 내용은 브라우저 저장소 할당량 및 제거 기준을 참조하십시오.

이 문서는 사이트의 사용 가능한 저장소를 추정하는 데 사용되는 {{domxref("StorageManager")}} 인터페이스에 대한 개요도 제공합니다.

## 개념과 사용법

### Storage buckets

저장소 표준에 설명된 저장소 시스템은 일반적으로 각 {{Glossary("origin")}}에 대해 단일 _버킷_으로 구성됩니다.

본질적으로 모든 웹사이트는 데이터가 배치되는 자체 저장 공간을 갖습니다. 그러나 경우에 따라 사용자 에이전트는 단일 origin의 데이터를 여러 다른 버킷에 저장하기로 결정할 수 있습니다. 예를 들어, 이 origin이 다른 서드파티 origin에 포함된 경우입니다.

자세한 내용은 브라우저가 다른 웹사이트의 데이터를 어떻게 분리하는가?를 참조하십시오.

This article gives an overview of the way user agents store and maintain websites' data. For more information about storage limits and eviction, see [Browser storage quotas and eviction criteria](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria).

This article also gives an overview of the {{domxref("StorageManager")}} interface used to estimate available storage for a site.

## 개념과 사용법

### Storage buckets

The storage system described by the Storage Standard, where site data is stored, usually consists of a single _bucket_ for each {{Glossary("origin")}}.

In essence, every website has its own storage space into which its data gets placed. In some cases however, user agents may decide to store a single origin's data in multiple different buckets, for example when this origin is embedded in different third-party origins.

To learn more, see [How browsers separate data from different websites?](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria#how_browsers_separate_data_from_different_websites)

### Bucket modes

Each site storage bucket has a _mode_ that describes the data retention policy for that bucket. There are two modes:

- `"best-effort"`
  - : The user agent will try to retain the data contained in the bucket for as long as it can, _but will not warn users_ if storage space runs low and it becomes necessary to clear the bucket in order to relieve the storage pressure.
- `"persistent"`
  - : The user agent will retain the data as long as possible, clearing all `"best-effort"` buckets before considering clearing a bucket marked `"persistent"`. If it becomes necessary to consider clearing persistent buckets, the user agent will notify the user and provide a way to clear one or more persistent buckets as needed.

You can change an origin's storage bucket mode by using the {{domxref("StorageManager.persist", "navigator.storage.persist()")}} method, which requires the `"persistent-storage"` [user permission](/ko/docs/Web/API/Permissions_API).

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

You can also use the {{domxref("StorageManager.persisted", "navigator.storage.persisted()")}} method to know whether an origin's storage is persistent or not:

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

To learn more, see [Does browser-stored data persist?](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria#does_browser-stored_data_persist).

### Quotas and usage estimates

The user agent determines, using whatever mechanism it chooses, the maximum amount of storage a given site can use. This maximum is the origin's **quota**. The amount of this space which is in use by the site is called its **usage**. Both of these values are estimates; there are several reasons why they're not precise:

- User agents are encouraged to obscure the exact size of the data used by a given origin, to prevent these values from being used for [fingerprinting](/ko/docs/Glossary/Fingerprinting) purposes.
- De-duplication, compression, and other methods to reduce the physical size of the stored data may be used.
- Quotas are conservative estimates of the space available for the origin's use, and should be less than the available space on the device to help prevent overruns.

To determine the estimated quota and usage values for a given origin, use the {{domxref("StorageManager.estimate", "navigator.storage.estimate()")}} method, which returns a promise that, when resolved, receives an object that contains these figures. For example:

```js
navigator.storage.estimate().then((estimate) => {
  // estimate.quota is the estimated quota
  // estimate.usage is the estimated number of bytes used
});
```

For more information about how much data an origin can store, see [How much data can be stored?](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria#how_much_data_can_be_stored).

### Data eviction

Data eviction is the process by which a user agent deletes an origin's stored data. This can happen, for example, when the device used to store the data is running low on storage space.

When clearing the data stored by an origin, the origin's bucket is treated as a single entity. The entire data stored by this origin is cleared.

If a bucket is marked as `"persistent"`, the contents won't be cleared by the user agent without either the data's origin itself or the user specifically doing so. This includes scenarios such as the user selecting a "Clear Caches" or "Clear Recent History" option. The user will be asked specifically for permission to remove persistent site storage buckets.

To learn more, see [When is data evicted?](/ko/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria#when_is_data_evicted).

## Interfaces

- {{domxref("StorageManager")}}
  - : Provides an interface for managing persistence permissions and estimating available storage.

### Extensions to other interfaces

- {{domxref("Navigator.storage")}} {{ReadOnlyInline}}
  - : Returns the singleton {{domxref("StorageManager")}} object used for managing persistence permissions and estimating available storage on a site-by-site/app-by-app basis.
- {{domxref("WorkerNavigator.storage")}} {{ReadOnlyInline}}
  - : Returns a {{domxref("StorageManager")}} interface for managing persistence permissions and estimating available storage.

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}

## See also

- [Using the Permissions API](/ko/docs/Web/API/Permissions_API/Using_the_Permissions_API)
- [Storage for the web on web.dev](https://web.dev/articles/storage-for-the-web)
- [Persistent storage on web.dev](https://web.dev/articles/persistent-storage)
- [Chrome Web Storage and Quota Concepts](https://docs.google.com/document/d/19QemRTdIxYaJ4gkHYf2WWBNPbpuZQDNMpUVf8dQxj4U/edit)
