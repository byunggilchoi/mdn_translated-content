---
title: Storage quotas and eviction criteria
slug: Web/API/Storage_API/Storage_quotas_and_eviction_criteria
page-type: guide
---

{{DefaultAPISidebar("Storage")}}

웹 개발자는 다양한 기술을 사용하여 사용자의 브라우저(예를 들어, 사용자가 웹 사이트를 보는 데 사용하는 장치의 로컬 디스크)에 데이터를 저장할 수 있습니다.

웹사이트가 브라우저에 저장할 수 있는 데이터의 양과 해당 한도에 도달했을 때 데이터를 삭제하는 데 사용하는 메커니즘은 브라우저마다 다릅니다.

이 문서에서는 데이터를 저장하는 데 사용할 수 있는 웹 기술, 웹 사이트에서 너무 많은 데이터를 저장하지 못하도록 제한하기 위해 브라우저가 설정한 할당량, 필요할 때 데이터를 삭제하는 데 사용하는 메커니즘에 대해 설명합니다.

## 브라우저가 다른 웹사이트의 데이터를 어떻게 분리합니까?

브라우저는 웹을 넘어 사용자가 추적될 위험을 줄이기 위해 버킷이라고도 하는 다양한 위치에 웹사이트의 데이터를 저장합니다. 대부분의 경우 브라우저는 _origin 별로_ 저장된 데이터를 관리합니다.

따라서 _{{Glossary("origin")}}_이라는 용어는 이 문서를 이해하는 데 중요합니다. origin은 스키마(예: HTTPS), 호스트네임 그리고 포트로 정의됩니다. 예를 들어 `https://example.com`과 `https://example.com/app/index.html`은 동일한 스키마(`https`), 호스트네임(`example.html`) 그리고 기본 포트를 갖기 때문에 동일한 origin에 속합니다.

이 문서에 설명된 할당량 및 제거 기준은 `https://example.com/site1/` 및 `https://example.com/site2`와 같이 여러 웹사이트를 실행하는 데 하나의 origin이 사용되는 경우에도 전체 origin에 적용됩니다.

그러나 경우에 따라 브라우저는 하나의 origin이 저장한 데이터를 여러 다른 파티션에 분리하기로 결정할 수도 있습니다. 예를 들어 여러 써드파티 origin들의 {{HTMLElement('iframe')}} 요소 내에 한 origin이 불러지는 경우입니다. 그러나 단순화를 위해 이 문서에서는 데이터가 항상 origin 별로 저장된다고 가정합니다.

## 브라우저에 데이터를 저장하는 기술은 무엇입니까?

웹 개발자는 다음 웹 기술을 사용하여 브라우저에 데이터를 저장할 수 있습니다:

| 기술                                                                                                 | 설명                                                                                                                                                                                                                       |
| --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Cookies](/en-US/docs/Web/HTTP/Cookies)                                                             | An HTTP cookie is a small piece of data that the web server and browser send each other to remember stateful information across page navigation.                                                                                  |
| [Web Storage](/en-US/docs/Web/API/Web_Storage_API)                                                  | The Web Storage API provides mechanisms for webpages to store string-only key/value pairs, including [`localStorage`](/en-US/docs/Web/API/Window/localStorage) and [`sessionStorage`](/en-US/docs/Web/API/Window/sessionStorage). |
| [IndexedDB](/en-US/docs/Web/API/IndexedDB_API)                                                      | IndexedDB is a Web API for storing large data structures in the browser and indexing them for high-performance searching.                                                                                                         |
| [Cache API](/en-US/docs/Web/API/Cache)                                                              | The Cache API provides a persistent storage mechanism for HTTP request and response object pairs that's used to make webpages load faster.                                                                                        |
| [Origin Private File System (OPFS)](/en-US/docs/Web/API/File_System_API/Origin_private_file_system) | OPFS provides a file system that's private to the origin of the page and can be used to read and write directories and files.                                                                                                     |

Note that, in addition to the above, browsers will store other types of data in the browser for an origin, such as [WebAssembly](/en-US/docs/WebAssembly) code caching.

## 브라우저에 저장된 데이터가 유지됩니까?

Data for an origin can be stored in two ways in a browser, _persistent_ and _best-effort_:

- Best-effort: this is the way that data is stored by default. Best-effort data persists as long as the origin is below its quota, the device has enough storage space, and the user doesn't choose to delete the data via their browser's settings.
- Persistent: an origin can opt-in to store its data in a persistent way. Data stored this way is only evicted, or deleted, if the user chooses to, by using their browser's settings. To learn more, see [When is data evicted](#when_is_data_evicted).

The data stored in the browser by an origin is best-effort by default. When using web technologies such as IndexedDB or Cache, the data is stored transparently without asking for the user's permission. Similarly, when the browser needs to evict best-effort data, it does so without interrupting the user.

If, for any reason, developers need persistent storage (e.g., when building a web app that relies on critical data that isn't persisted anywhere else), they can do so by using the {{domxref("StorageManager.persist()", "navigator.storage.persist()")}} method of the {{domxref("Storage_API", "Storage API", "", "nocode")}}.

In Firefox, when a site chooses to use persistent storage, the user is notified with a UI popup that their permission is requested.

Safari and most Chromium-based browsers, such as Chrome or Edge, automatically approve or deny the request based on the user's history of interaction with the site and do not show any prompts to the user.

Note that [research from the Chrome team](https://web.dev/articles/persistent-storage) shows that data is very rarely deleted by the browser. If a user visits a website regularly, there is very little chance that its stored data, even in best-effort mode, will get evicted by the browser.

### Private browsing

Note that in private browsing mode (also called _Incognito_ in Chrome, and _InPrivate_ in Edge), browsers may apply different quotas, and stored data is usually deleted when the private browsing mode ends.

## 얼마나 많은 데이터를 저장할 수 있습니까?

### Cookies

Different browsers have different rules around how many cookies are allowed per origin and how much space these cookies can use on the disk. While cookies are useful for preserving some small shared state between the browser and the web server across page navigation, using cookies for storing data in the browser is not advised. Cookies are sent with each and every HTTP request, so storing data in cookies that could be stored by using another web technology unnecessarily increases the size of requests.

Because cookies should not be used for storing data in the browser, cookie storage browser limits are not covered here.

### Web Storage

Web Storage, which can be accessed by using the {{domxref("Window.localStorage", "localStorage")}} and {{domxref("Window.sessionStorage", "sessionStorage")}} properties of the {{domxref("window")}} object, is limited to 10 MiB of data maximum on all browsers.

Browsers can store up to 5 MiB of local storage, and 5 MiB of session storage per origin.

Once this limit is reached, browsers throw a `QuotaExceededError` exception which should be handled by using a {{jsxref("Statements/try...catch","try...catch")}} block.

### Other web technologies

The data that's stored by using other web technologies, such as IndexedDB, Cache API, or File System API (which defines the Origin Private File System), is managed by a storage management system that's specific to each browser.

This system regulates all of the data that an origin stores using these APIs.

Each browser determines, using whatever mechanism it chooses, the maximum amount of storage a given origin can use.

#### Firefox

In Firefox, the maximum storage space an origin can use in best-effort mode is whichever is the smaller of:

- 10% of the total disk size where the profile of the user is stored.
- Or 10 GiB, which is the _group limit_ that Firefox applies to all origins that are part of the same {{Glossary("eTLD", "eTLD+1 domain")}}.

Origins for which persistent storage has been granted can store up to 50% of the total disk size, capped at 8 TiB, and are not subject to the eTLD+1 group limit.

For example, if the device has a 500 GiB hard drive, Firefox will allow an origin to store up to:

- In best-effort mode: 10 GiB of data, which is the eTLD+1 group limit.
- In persistent mode: 250 GiB, which is 50% of the total disk size.

Note that it might not actually be possible for the origin to reach its quota because it is calculated based on the hard drive **total** size, not the currently available disk space. This is done for security reasons, to avoid {{Glossary("fingerprinting")}}.

#### Chrome and Chromium-based browsers

In browsers based on the [Chromium open-source project](https://www.chromium.org/Home/), including Chrome and Edge, an origin can store up to 60% of the total disk size in both persistent and best-effort modes.

For example, if the device has a 1 TiB hard drive, the browser will allow an origin to use up to 600 GiB.

Like with Firefox, because this quota is calculated based on the hard drive total size to avoid fingerprinting, an origin might not actually be able to reach its quota.

#### Safari

Starting with macOS 14 and iOS 17, Safari allots up to around 20% of the total disk space for each origin. If the user has saved it as a web app on the Home Screen or the Dock, this limit is increased to up to 60% of the disk size. For privacy reasons, {{Glossary("Same-origin policy", "cross-origin")}} frames have a separate quota, amounting to roughly 1/10 of their parents.

For instance, a macOS device with a 1 TiB drive will limit each origin to around 200 GiB. If the user stores a web app on its Dock, that will be alloted a greater limit of around 600 GiB.

Like other browsers, the exact limits enforced by the quota may vary as to avoid fingerprinting. Additionally, Safari also enforces an overall quota that stored data across all origins cannot grow beyond: 80% of disk size for each browser and web app, and 15% of disk size for each non-browser app that displays web content. More info on Safari's storage policies can be found on the [Webkit blog](https://www.webkit.org/blog/14403/updates-to-storage-policy/).

In earlier versions of Safari, an origin is given an initial 1 GiB quota. Once the origin reaches this limit, Safari asks the user for permission to let the origin store more data. This happens whether the origin stores data in best-effort mode or persistent mode.

## How to check the available space?

Web developers can check how much space is available for their origin and how much is being used by the origin with the {{domxref("StorageManager.estimate()", "navigator.storage.estimate()")}} method of the {{domxref("Storage_API", "Storage API", "", "nocode")}}.

Note that this method only returns the estimated usage value, not the actual value. Some of the resources that are stored by an origin may be coming from other origins and browsers voluntarily pad the size of the cross-origin data when reporting total usage value.

## What happens when an origin fills its quota?

Attempting to store more than an origin's quota using IndexedDB, Cache, or OPFS, for example, fails with a `QuotaExceededError` exception.

Web developers should wrap JavaScript that writes to browser storage within {{jsxref("Statements/try...catch","try...catch")}} blocks. Freeing up space by deleting data before storing new data is also recommended.

## 데이터는 언제 제거됩니까?

Data eviction is the process by which a browser deletes an origin's stored data.

Data eviction can happen in multiple cases:

- When the device is running low on storage space, also known as _storage pressure_.
- When all of the data stored in the browser (across all origins) exceeds the total amount of space the browser is willing to use on the device.
- Proactively, for origins that aren't used regularly, which happens only in Safari.

### Storage pressure eviction

When a device is running low on storage space, also known as _storage pressure_, there may come a point when the browser has less available space than it needs to store all of the origin's stored data.

Browsers use a Least Recently Used (LRU) policy to deal with this scenario. The data from the least recently used origin is deleted. If storage pressure continues, the browser moves on to the second least recently used origin, and so on, until the problem is resolved.

This eviction mechanism only applies to origins that are not persistent and skips over origins that have been granted data persistence by using {{domxref("StorageManager.persist()", "navigator.storage.persist()")}}.

### Browser maximum storage exceeded eviction

Some browsers define a maximum storage space that they can use on the device's hard disk. For example, Chrome currently uses at most 80% of the total disk size.

This maximum storage size means that there may come a point at which the data stored by all of the combined origins exceeds the maximum size without any one origin being above its individual quota.

When this happens, the browser starts evicting best-effort origins as described in [Storage pressure eviction](#storage_pressure_eviction).

### Proactive eviction

Safari proactively evicts data when cross-site tracking prevention is turned on. If an origin has no user interaction, such as click or tap, in the last seven days of browser use, its data created from script will be deleted. Cookies set by server are exempt from this eviction.

## How is data evicted?

When an origin's data is evicted by the browser, all of its data, not parts of it, is deleted at the same time. If the origin had stored data by using IndexedDB and the Cache API for example, then both types of data are deleted.

Only deleting some of the origin's data could cause inconsistency problems.

## See also

- [Storage for the web on web.dev](https://web.dev/articles/storage-for-the-web)
- [Persistent storage on web.dev](https://web.dev/articles/persistent-storage)
- [Chrome Web Storage and Quota Concepts](https://docs.google.com/document/d/19QemRTdIxYaJ4gkHYf2WWBNPbpuZQDNMpUVf8dQxj4U/edit)
