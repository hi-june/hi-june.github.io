---
title:  "[자바 코테] java도 itertools가 있었으면 좋겠다.."
excerpt: "코딩 테스트용 자바 정리"

categories:
  - Java Coding Test
tags:
  - java

toc: true
toc_sticky: true

date: 2023-04-06
last_modified_at: 2023-05-06
---
## 가끔 파이썬이 쓰고싶어지는 순간이 있다..
자바에는 아쉽게도 파이썬의 `itertools`와 같은 라이브러리가 없다.  
따라서 순열, 조합 관련된 코드들은 직접 구현해서 사용해야한다.  

## 조합
```java
import java.util.*;

class Combination<T> {
    private int n;
    private int r;
    private int[] now;
    private List<List<T>> result;

    public List<List<T>> getResult() {
        return result;
    }

    public Combination(int n, int r) {
        this.n = n;
        this.r = r;
        this.now = new int[r];
        this.result = new ArrayList<>();
    }

    public void combination(List<T> list, int depth, int index, int target) {
        if (depth == r) {
            List<T> temp = new ArrayList<>();
            for (int i = 0; i < now.length; i++) {
                temp.add(list.get(now[i]));
            }
            result.add(temp);
            return;
        }
        if (target == n) return;
        now[index] = target;
        combination(list, depth + 1, index + 1, target + 1);
        combination(list, depth, index, target + 1);
    }
}

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Tom", "Jimmy", "Harry");
        int r = 2;

        Combination comb = new Combination(list.size(), r);
        comb.combination(list, 0, 0, 0);
        List<List<String>> result = comb.getResult();

        System.out.println(result);
    }
}
```

```shell
# Combination result
[[Tom, Jimmy], [Tom, Harry], [Jimmy, Harry]]
```


## 중복 조합
```java
import java.util.*;

class CombinationWithRepetition<T> {
    private int n;
    private int r;
    private int[] now;
    private List<List<T>> result;

    public List<List<T>> getResult() {
        return result;
    }

    public CombinationWithRepetition(int n, int r) {
        this.n = n;
        this.r = r;
        this.now = new int[r];
        this.result = new ArrayList<>();
    }

    public void combination(List<T> list, int depth, int index) {
        if (depth == r) {
            List<T> temp = new ArrayList<>();
            for (int i = 0; i < now.length; i++) {
                temp.add(list.get(now[i]));
            }
            result.add(temp);
            return;
        }

        for (int i = index; i < n; i++) {
            now[depth] = i;
            combination(list, depth + 1, i);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Tom", "Jimmy", "Harry");
        int r = 2;

        CombinationWithRepetition comb = new CombinationWithRepetition(list.size(), r);
        comb.combination(list, 0, 0);
        List<List<String>> result = comb.getResult();

        System.out.println(result);
    }
}
```

```shell
# CombinationWithRepetition result
[[Tom, Tom], [Tom, Jimmy], [Tom, Harry], [Jimmy, Jimmy], [Jimmy, Harry], [Harry, Harry]]
```


## 순열
```java
import java.util.*;

class Permutation<T> {
    private int n;
    private int r;
    private int[] now;
    private boolean[] visited;
    private List<List<T>> result;

    public List<List<T>> getResult() {
        return result;
    }

    public Permutation(int n, int r) {
        this.n = n;
        this.r = r;
        this.now = new int[r];
        this.visited = new boolean[n];
        this.result = new ArrayList<>();
    }

    public void permutation(List<T> list, int depth) {
        if (depth == r) {
            List<T> temp = new ArrayList<>();
            for (int i = 0; i < now.length; i++) {
                temp.add(list.get(now[i]));
            }
            result.add(temp);
            return;
        }

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                visited[i] = true;
                now[depth] = i;
                permutation(list, depth + 1);
                visited[i] = false;
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Tom", "Jimmy", "Harry");
        int r = 2;

        Permutation perm = new Permutation(list.size(), r);
        perm.permutation(list, 0);
        List<List<String>> result = perm.getResult();

        System.out.println(result);
    }
}
```

```shell
# Permutation result
[[Tom, Jimmy], [Tom, Harry], [Jimmy, Tom], [Jimmy, Harry], [Harry, Tom], [Harry, Jimmy]]
```


## 중복 순열
```java
import java.util.*;

class PermutationWithRepetition<T> {
    private int n;
    private int r;
    private int[] now;
    private List<List<T>> result;

    public List<List<T>> getResult() {
        return result;
    }

    public PermutationWithRepetition(int n, int r) {
        this.n = n;
        this.r = r;
        this.now = new int[r];
        this.result = new ArrayList<>();
    }

    public void permutation(List<T> list, int depth) {
        if (depth == r) {
            List<T> temp = new ArrayList<>();
            for (int i = 0; i < now.length; i++) {
                temp.add(list.get(now[i]));
            }
            result.add(temp);
            return;
        }

        for (int i = 0; i < n; i++) {
            now[depth] = i;
            permutation(list, depth + 1);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Tom", "Jimmy", "Harry");
        int r = 2;

        PermutationWithRepetition perm = new PermutationWithRepetition(list.size(), r);
        perm.permutation(list, 0);
        List<List<String>> result = perm.getResult();

        System.out.println(result);
    }
}
```

```shell
# PermutationWithRepetition result
[[Tom, Tom], [Tom, Jimmy], [Tom, Harry], [Jimmy, Tom], [Jimmy, Jimmy], [Jimmy, Harry], [Harry, Tom], [Harry, Jimmy], [Harry, Harry]]
```