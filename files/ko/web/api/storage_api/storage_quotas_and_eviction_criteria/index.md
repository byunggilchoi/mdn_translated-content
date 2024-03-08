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
| [Cookies](/ko/docs/Web/HTTP/Cookies)                                                             | HTTP 쿠키는 웹 서버와 브라우저가 페이지 탐색 전반에 걸쳐 상태 정보를 저장하기 위해 서로에게 보내는 작은 데이터 조각입니다.                                                                                |
| [Web Storage](/ko/docs/Web/API/Web_Storage_API)                                                  | Web Storage API는 웹페이지가 [`localStorage`](/ko/docs/Web/API/Window/localStorage) 및 [`sessionStorage`](/ko/docs/Web/API/Window/sessionStorage)를 포함하여 문자열 전용 키/값 쌍을 저장하는 방법을 제공합니다. |
| [IndexedDB](/ko/docs/Web/API/IndexedDB_API)                                                      | IndexedDB는 대용량 데이터 구조를 브라우저에 저장하고 고성능 검색을 위해 인덱싱하는 웹 API입니다.                                                                                                         |
| [Cache API](/ko/docs/Web/API/Cache)                                                              | Cache API는 웹페이지를 더 빠르게 로드하기 위한 HTTP 요청 및 응답 객체 쌍을 영구적으로 저장할 수 있는 방법을 제공합니다.                                                                                        |
| [Origin Private File System (OPFS)](/ko/docs/Web/API/File_System_API/Origin_private_file_system) | OPFS는 페이지의 origin에 대한 비공개 파일 시스템을 제공하며 디렉터리와 파일을 읽고 쓰기 위해 사용할 수 있습니다.                                                                                                     |

위의 기술들 외에도 브라우저는 [WebAssembly](/ko/docs/WebAssembly) 코드 캐싱과 같은 origin이 가진 다른 타입의 데이터도 브라우저에 저장합니다.

## 브라우저에 저장된 데이터가 유지됩니까?

한 origin의 데이터는 _persistent_, _best-effort_ 두 가지 방법으로 브라우저에 저장됩니다:

- Best-effort: 기본적으로 데이터가 저장되는 방식입니다. origin이 할당량 미만을 쓰고 있고, 기기에 충분한 저장 공간이 있고, 사용자가 브라우저 설정을 통해 데이터 삭제를 선택하지 않는 한 Best-effort 상태의 데이터는 유지됩니다.
- Persistent: 어떤 origin에 대해서 데이터를 persistent 방식으로 저장하게 할 수도 있습니다. 이 방식으로 저장된 데이터는 사용자가 브라우저 설정을 사용하여 선택한 경우에만 제거되거나 삭제됩니다. 자세한 내용은 [데이터는 언제 제거됩니까?](#데이터는_언제_제거됩니까)를 참조하세요.

origin에 의해 브라우저에 저장된 데이터는 기본적으로 best-effort 모드를 적용합니다. 이 상태에서 IndexedDB나 Cache 등의 웹 기술을 사용하면 사용자의 허락을 구하지 않고 데이터가 저장됩니다. 마찬가지로, 브라우저가 best-effort 상태의 데이터를 제거해야 할 때도 사용자를 방해하지 않고 제거합니다.

어떤 이유로든 개발자에게 영구 저장소가 필요한 경우(예: 다른 곳에서는 유지되지 않는 중요한 데이터에 의존하는 웹 앱을 구축하는 경우) {{domxref("Storage_API", "Storage API", "", "nocode")}}의  {{domxref("StorageManager.persist()", "navigator.storage.persist()")}} 메서드를 사용할 수 있습니다.

Firefox에서는 사이트가 영구 저장소를 사용하기로 선택하면 사용자에게 권한이 요청된다는 UI 팝업이 표시됩니다.

Safari 및 Chrome이나 Edge와 같은 대부분의 Chromium 기반 브라우저는 사용자의 사이트 상호 작용 기록을 기반으로 요청을 자동으로 승인하거나 거부하며 사용자에게 어떠한 메시지도 표시하지 않습니다.

[Chrome 팀의 연구](https://web.dev/articles/pertant-storage)에 따르면 브라우저에서 데이터가 삭제되는 경우는 거의 없습니다. 사용자가 웹 사이트를 정기적으로 방문하는 경우 best-effort 모드에서도 저장된 데이터가 브라우저에 의해 제거될 가능성은 거의 없습니다.

### 비공개 브라우징

비공개 브라우징 모드(Chrome에서는 _Incognito_, Edge에서는 _InPrivate_라고도 함)에서는 브라우저가 다른 할당량을 적용할 수 있으며 일반적으로 비공개 브라우징 모드가 종료되면 저장된 데이터가 삭제됩니다.

## 얼마나 많은 데이터를 저장할 수 있습니까?

### Cookies

브라우저마다 한 origin에 허용되는 쿠키 수와 그 쿠키가 디스크에서 사용할 수 있는 용량에 대한 규칙이 다릅니다. 쿠키는 페이지들을 탐색할 때 브라우저와 웹 서버 간의 일부 작은 상태값들을 공유하는데 유용하지만 브라우저에 데이터를 저장하기 위해 쿠키를 사용하는 것은 권장되지 않습니다. 쿠키는 모든 HTTP 리퀘스트에 함께 전송되므로 다른 웹 기술을 사용하여 저장할 수 있는 데이터를 쿠키에 저장하면 리퀘스트의 크기가 불필요하게 늘어납니다.

쿠키는 브라우저에 데이터를 저장할 때 사용되면 안 되기 때문에 쿠키 저장에 대한 각 브라우저들의 규칙은 여기에서 다루지 않습니다.

### 웹 스토리지

웹 스토리지는 모든 브라우저에서 최대 10MiB의 데이터로 제한되며 {{domxref("window")}} 객체의 속성들인 {{domxref("Window.localStorage", "localStorage")}} 및 {{domxref("Window.sessionStorage", "sessionStorage")}}를 사용하여 액세스할 수 있습니다.

브라우저는 origin 하나 당 최대 5MiB의 로컬 스토리지와 5MiB의 세션 스토리지를 저장할 수 있습니다.

이 제한에 도달하면 브라우저는 {{jsxref("Statements/try...catch","try...catch")}} 블록을 사용하여 처리해야 하는 `QuotaExceededError` 예외를 발생시킵니다.

### 기타 웹 기술

IndexedDB, Cache API 또는 파일 시스템 API(Origin 개인 파일 시스템 정의)와 같은 다른 웹 기술을 사용하여 저장된 데이터는 각 브라우저의 특정 저장소 관리 시스템에 의해 관리됩니다.

이 시스템은 이러한 API를 사용하여 origin이 저장하는 모든 데이터를 제한합니다.

각 브라우저는 각자의 메커니즘을 사용하여 특정 origin이 사용할 수 있는 최대 저장 용량을 결정합니다.

#### Firefox

Firefox에서 best-efford 모드인 origin이 사용할 수 있는 최대 저장 용량은 다음 두 가지 중 더 작은 값입니다.

- 사용자 프로필이 저장된 전체 디스크 크기의 10%입니다.
- 아니면 Firefox가 같은 {{Glossary("eTLD", "eTLD+1 domain")}}에 해당하는 origin 전체에 적용하는 _그룹 한도_인 10GiB입니다.

persistent 모드인 origin은 최대 8TiB 범위 내에서 총 디스크 크기의 최대 50%를 사용할 수 있으며 eTLD+1 그룹 제한이 적용되지 않습니다.

예를 들어 장치에 500GiB 하드 드라이브가 있는 경우 Firefox에서 origin은 다음과 같이 저장할 수 있습니다.

- best-effort 모드: 10 GiB의 데이터, 이는 eTLD+1 그룹 한도입니다.
- persistent 모드: 250 GiB, 이는 전체 디스크 크기의 50%입니다.

origin이 할당량까지 사용하는 것은 실제로 불가능할 수도 있습니다. 할당량은 현재 사용 가능한 디스크 공간이 아닌 하드 드라이브의 **총** 크기를 기준으로 계산되기 때문입니다. {{Glossary("fingerprinting")}}을 방지하기 위한 보안상의 이유 때문입니다.

#### Chrome 및 Chromium 기반 브라우저

Chrome 및 Edge를 포함한 [Chromium 오픈 소스 프로젝트](https://www.chromium.org/Home/) 기반 브라우저에서 한 origin이 persistent, best-effort 모드 모두 전체 디스크 크기의 최대 60%를 이용할 수 있습니다.

예를 들어 장치에 1TiB 하드 드라이브가 있는 경우 한 origin이 최대 600GiB를 사용할 수 있습니다.

Firefox와 마찬가지로 이 할당량은 핑거프린팅 방지를 위해 하드 드라이브 전체 크기를 기준으로 계산되므로 실제로 이 할당량까지 쓰지 못할 수도 있습니다.

#### Safari

macOS 14와 iOS 17부터 Safari는 전체 디스크 공간의 약 20%를 각 origin에 할당합니다. 사용자가 홈 화면이나 dock에 웹 앱으로 저장한 경우 디스크 크기의 최대 60%까지 제한이 늘어납니다. 개인정보 보호를 위해 {{Glossary("Same-origin policy", "cross-origin")}} 창에는 상위 창의 대략 1/10 정도를 별도로 할당합니다.

예를 들어 1TiB 드라이브가 있는 macOS 장치는 각 원본을 약 200GiB로 제한합니다. 사용자가 Dock에 웹 앱을 저장하는 경우 약 600GiB까지 제한이 풀립니다.

다른 브라우저와 마찬가지로 핑거프린팅을 방지하기 위해 할당량에 적용되는 정확한 제한방법은 다를 수 있습니다. 또한 Safari는 모든 origin 통틀어 저장된 데이터가 각 브라우저 및 웹 앱의 경우에는 디스크 크기의 80%, 웹 콘텐츠를 표시하는 비 브라우저 앱의 경우 디스크 크기의 15%를 초과할 수 없도록 제한을 적용합니다. Safari의 저장소 정책에 대한 자세한 내용은 [Webkit 블로그](https://www.webkit.org/blog/14403/updates-to-storage-policy/)에서 확인할 수 있습니다.

이전 버전의 Safari에서는 한 origin에 1GiB 할당량을 기본으로 제공됩니다. 특정 origin이 이 제한에 도달하면 Safari는 사용자에게 이 origin에 더 많은 데이터를 저장할 수 있도록 허가를 요청합니다. 이는 origin의 모드가 best-effort든 persistent이든 관계없이 요청됩니다.

## 사용 가능한 공간을 어떻게 확인합니까?

웹 개발자는 {{domxref("Storage_API", "Storage API", "", "nocode")}}의 {{domxref("StorageManager.estimate()", "navigator.storage.estimate()")}} 메서드를 사용하여 origin이 사용 가능한 용량과 사용 중인 용량을 확인할 수 있습니다.

이 메서드는 실제 값이 아닌 예상 사용량 값만 반환합니다. 한 origin에 의해 저장된 리소스 중 일부는 다른 origin에서 올 수도 있으며 브라우저는 origin 사이에 교차된 데이터의 크기만큼 숫자를 키워서 총 사용량 값을 보고합니다.

## origin이 할당량을 다 채우면 어떻게 됩니까?

예를 들어 IndexedDB, Cache 또는 OPFS를 사용하여 origin에 할당된 양보다 많은 양을 저장하려고 시도하면 `QuotaExceededError`를 반환하고 실패하게 됩니다.

웹 개발자는 {{jsxref("Statements/try...catch","try...catch")}} 블록 내에 브라우저 저장소에 쓰는 JavaScript를 넣어야 합니다. 새로운 데이터를 저장하기 전에 데이터를 삭제하여 공간을 확보하는 것을 추천합니다.

## 데이터는 언제 제거됩니까?

데이터 제거는 브라우저가 origin이 저장한 데이터를 삭제하는 프로세스입니다.

데이터 제거는 여러 경우에 발생할 수 있습니다:

- 장치의 저장 공간이 부족한 경우로 _저장 압력_이 있는 경우.
- 브라우저에 저장된 (모든 origin에 걸쳐서) 모든 데이터가 브라우저가 기기에서 사용하려는 총 공간을 초과하는 경우.
- Safari에서는 정기적으로 사용되지 않는 originㄹ에 대해 사전에 제거합니다.

### 저장 압력에 따른 제거

장치의 저장 공간이 부족한 경우를 _저장 압력_이라고도 하는데, 언젠가는 모든 origin들의 데이터를 저장하기에는 브라우저의 사용 가능한 공간이 부족한 시점이 올 수 있습니다.

브라우저는 LRU(Least Recent Used) 정책을 사용하여 이 시나리오를 처리합니다. 최근에 가장 적게 사용된 origin의 데이터가 삭제됩니다. 저장 공간 부족이 계속되면 브라우저는 문제가 해결될 때까지 그 다음으로 최근에 사용되지 않은 origin을 삭제허려고 할 것입니다.

이 제거 메커니즘은 persistent가 아닌 origin에만 적용되며 {{domxref("StorageManager.persist()", "navigator.storage.persist()")}}를 사용하여 persistent 모드가 부여된 origin들은 건너뜁니다.

### 브라우저 최대 저장 용량 초과에 따른 제거

일부 브라우저는 최대 저장 공간을 기기의 하드 디스크에서 사용할 수 있는 용량을 통해 정의합니다. 예를 들어 Chrome은 현재 전체 디스크 크기의 최대 80%를 최대 저장 공간 값으로 사용합니다.

이 최대 저장 용량은 단일 오리진이 개별 할당량을 초과하지 않고 결합된 모든 오리진에 의해 저장된 데이터가 최대 크기를 초과하는 지점이 발생할 수 있음을 의미합니다.

이런 일이 발생하면 브라우저는 [저장 압력 제거](#저장_압력에_따른_제거)에 설명된 대로 best-effort 모드인 origin들부터 제거하기 시작합니다.

### 사전적 제거

Safari는 교차 사이트 추적 방지 기능이 켜져 있으면 사전에 데이터를 제거합니다. 지난 7일 동안 브라우저를 사용하면서 해당 origin에 클릭이나 탭과 같은 사용자 상호 작용이 없었으면 스크립트에서 생성된 데이터가 삭제됩니다. 서버에서 설정한 쿠키는 이 제거 규칙에서 제외됩니다.

## 데이터는 어떻게 제거됩니까?

origin의 데이터가 브라우저에 의해 제거되면 해당 데이터의 일부가 아닌 모든 데이터가 동시에 삭제됩니다. 예를 들어 origin이 IndexedDB 및 Cache API를 사용하여 데이터를 저장한 경우 두 가지 유형의 데이터가 모두 삭제됩니다.

원본 데이터 중 일부만 삭제하면 불일치 문제가 발생할 수 있습니다.

## 같이 보기

- [Storage for the web on web.dev](https://web.dev/articles/storage-for-the-web)
- [Persistent storage on web.dev](https://web.dev/articles/persistent-storage)
- [Chrome Web Storage and Quota Concepts](https://docs.google.com/document/d/19QemRTdIxYaJ4gkHYf2WWBNPbpuZQDNMpUVf8dQxj4U/edit)
