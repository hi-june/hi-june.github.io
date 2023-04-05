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
last_modified_at: 2023-04-06
---
## 가끔 파이썬 마려운 순간이 있다..
자바에는 아쉽게도 파이썬의 `itertools`와 같은 라이브러리가 없다.  
따라서 순열, 조합 관련된 코드들은 직접 구현해서 사용해야한다.  

## 조합
```java
class Combination<T> {
    private int n;
    private int r;
    private int[] now; // 현재 조합
    private List<List<T>> result; // 모든 조합

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
    public static void main(String[] args) throws IOException {
        List<String> list = Arrays.asList("Tom", "Jimmy", "Harry");
        int r = 2;

        Combination comb = new Combination(list.size(), r);
        comb.combination(list, 0, 0, 0);
        List<List<String>> result = comb.getResult();

        System.out.println(result);
    }
}
```