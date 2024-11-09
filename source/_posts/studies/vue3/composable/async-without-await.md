---
title: Async Without Await
date: 2024-11-03 20:52:36
categories:
  - Studies
  - Vue3
  - Composable
#tags:
---
비동기 함수를 실행하기 전과 후에 관련 reactive value들을 동기적으로 업데이트하는 패턴입니다.

`.vue` 파일에 `async`/`await`문 없이 비동기 작업 로직을 간결하게 작성할 수 있습니다.

```js
export default useMyAsyncComposable(promise) {
  const state = ref(null);
  const execute = async () => {
    // 2. Waiting for the promise to finish
    state.value = await promise
    // 5. Sometime later...
    // Promise has finished, `state` is updated reactively,
    // and we finish this method
  }
  // 1. Run the `execute` method
  execute();
  // 3. The `await` returns control to this point
  // 4. Return state and continue with the `setup` function
  return state;
}
```

이와 관련된 VueUse hook으로 [useAsycState](https://vueuse.org/core/useAsyncState/#useasyncstate)가 있는데, 실제 소스의 간단한 버전은 다음과 같이 작성할 수 있습니다.

```js
export function useAsyncState(promise, initialState) {
  const state = ref(initialState);
  const isReady = ref(false);
  const isLoading = ref(false);
  const error = ref(undefined);

  async function execute() {
    error.value = undefined;
    isReady.value = false;
    isLoading.value = true;
    try {
      const data = await promise;
      state.value = data;
      isReady.value = true;
    } catch (e) {
      error.value = e;
    }
    isLoading.value = false;
  }

  execute();

  return {
    state,
    isReady,
    isLoading,
    error,
  };
}
```
