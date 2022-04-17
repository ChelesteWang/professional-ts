## TS 类型记录

定义 useState 类型

```ts
type StateType<T> = T | undefined;

const [teams, setTeams] = useState<StateType<ITeam[]>>();
```

定义 useAsyncDataEffect

```ts
export function useAsyncDataEffect<T>(
  getData: () => Promise<T>,
  options: {
    stateName: string;
    otherStatesToMonitor?: unknown[];
    setter: (arg: T) => void;
  },
): void {
  let cancelled = false;
  const { setter, stateName } = options;
  useEffect(() => {
    // Deferred 用于确保 promise 只被解析一次
    const d = new Deferred<T>();

    getData()
      .then((jsonData) => {
        if (cancelled) return;
        else d.resolve(jsonData);
      })
      .catch(d.reject);

    d.promise
      .then((data) => {
        if (!cancelled) {
          console.info(
            '%c Updating state: ' + stateName,
            'background: green; color: white; display: block;',
          );
          setter(data);
        }
      })
      .catch(console.error);
    return () => {
      cancelled = true;
    };
  }, [...(options.otherStatesToMonitor || []), stateName]);
}
```
