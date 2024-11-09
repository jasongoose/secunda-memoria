---
title: Scaling Up with Reducer and Context
date: 2024-11-04 08:18:01
categories:
  - Studies
  - React
  - Learn React
#tags:
---
`useReducer`와 `useContext`를 같이 사용하면 복잡한 비지니스 로직을 위한 상태관리 업데이트 방식을 통합시킬 수 있습니다.

`useReducer`는 이벤트 핸들러 로직을 간단하게 해주지만 관련 state와 dispath 함수는 상위 컴포넌트에만 존재하기 때문에 자손 컴포넌트에서 dispatch를 수행하려면 이 둘을 props pipe로 전달해야 합니다.

규모가 큰 앱에서는 props drilling 문제가 발생하므로 여기서 대안으로 reducer의 state와 dispatch 함수를 개별 context로 전달하면 됩니다.

```jsx
// TasksContext.js

import { createContext } from "react";

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

```jsx
export default function App() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Day off in Kyoto</h1>
        <AddTask />
        <TaskList />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

```jsx
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
}
```

```jsx
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
	);
}
```

여기서 reducer + context + Provider 컴포넌트를 하나의 파일에서 관리하면 컴포넌트는 데이터를 보여주기만 하는 Presenter로써 역할을 가지므로 간결한 소스를 작성할 수 있습니다.

```jsx
import { createContext, useContext, useReducer } from "react";

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
  // ...
}

const initialTasks = [
  { id: 0, text: "Philosopher’s Path", done: true },
  { id: 1, text: "Visit the temple", done: false },
  { id: 2, text: "Drink matcha", done: false },
];
```

```jsx
import { TasksProvider } from "./TasksContext.js";

export default function App() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

```jsx
import { useTasks } from "./TasksContext.js";

export default function TaskList() {
  const tasks = useTasks();
  // ...
}
```

```jsx
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();
  // ...
  return (
    // ...
	);
}
```
