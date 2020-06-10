---
title: "자료구조와 함께 배우는 알고리즘 입문 (4) 소수의 나열"
date: 2020-06-10 23:58:00 +09:00
categories: programming
tags: [algorithm, datastructure, programming, book, IT]
---

간단하게 2차원 배열을 선언하자면, `int[][] x = new int[2][4]` 과 같다.

Java는 엄밀한 의미에서 다차원 배열이 없다. 2차원 배열을 '배열의 배열'로 생각하고 3차원을 '배열의 배열의 배열'로 생각하기 때문이다. 따라서 배열 x의 형은 아래와 같다.

> int 형을 구성 자료형으로 하는 배열"을 구성 자료형으로 하는 배열

다차원 배열 개념을 연습해보기 위해 예시를 들어보자.

### 한 해의 경과 일 수를 나타내는 프로그램

```
package chap02;
import java.util.Scanner;
// 그 해 경과 일수를 구함

class DayOfYear {
    // 각 달의 일 수
    static int[][] mdays = {
        {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31},    // 평년
        {31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}    // 윤년
    };

    // 서기 year년은 윤년인가? (윤년：1／평년：0)
    static int isLeap(int year) {
        return (year % 4 == 0 && year % 100 != 0 || year % 400 == 0) ? 1 : 0;
    }

    // 서기 y년 m월 d일의 그 해 경과 일수를 구함
    static int dayOfYear(int y, int m, int d) {
        int days = d;                        // 일 수

        for (int i = 1; i < m; i++)            // 1월~(m-1)월의 일 수를 더함
            days += mdays[isLeap(y)][i - 1];
        return days;
    }

    public static void main(String[] args) {
        Scanner stdIn = new Scanner(System.in);
        int retry;                            // 다시 한 번?

        System.out.println("그 해 경과 일 수를 구합니다.");

        do {
            System.out.print("년：");  int year  = stdIn.nextInt();    // 년
            System.out.print("월：");  int month = stdIn.nextInt();    // 월
            System.out.print("일：");  int day   = stdIn.nextInt();    // 일

            System.out.printf("그 해 %d일째입니다.\n", dayOfYear(year, month, day));

            System.out.print("한번 더 할까요? (1.예／0.아니오）：");
            retry = stdIn.nextInt();
        } while (retry == 1);
    }
}
```

### 파생 연습 문제

1.  메서드 dayOfYear를 변수 i와 days 없이 구현하기. (while문 사용)
2.  y년 m월 d일의 그 해 남을 일 수(12월 31일이면 0, 12월 30일이면 1)를 구하는 아래 메서드를 작성하기.
    -   `static int leftDayOfYear(int y, int m, int d)`

### 배열에 관한 보충 설명

1.  빈 배열을 만들 수 있다.
2.  배열 요소의 접근
    -   배열의 접근은 모두 런타임(실행 시)에 검사된다. 만약 0 미만 또는 배열 요솟수 이상의 인덱스를 사용하면 IndexOutOfBoundsException이 발생한다.
3.  배열 초기화의 쉼표
    -   배열을 초기화할 때 마지막 요소에 대한 초기화 뒤에 추가로 쉼표를 넣을 수 있다.
4.  다차원 배열의 복제
    -   다차원 배열을 복제할 때는 최상위의 1레벨만 복제 되고 그 아래 레벨의 배열은 복제되지 않고 공유된다.
5.  확장 for문을 사용하여 배열의 스캔을 매우 간단하게 구현할 수 있다.
    -   배열의 모든 요소를 스캔하는 과정에서 인덱스 자체의 값이 필요하지 않으면 확장 for문을 이용하는 것이 좋다.
    -   확장 for문은 for-in문 또는 for-each문이라 불리기도 한다.

아래는 확장 for문 예시

```
package chap02;
// 배열의 모든 요소의 합을 구하여 출력함 (확장 for문)

class ArraySumForIn {
    public static void main(String[] args) {
        double[] a = { 1.0, 2.0, 3.0, 4.0, 5.0 };

        for (int i = 0; i < a.length; i++)
            System.out.println("a[" + i + "] = " + a[i]);

        double sum = 0;        // 합계
        for (double i : a)
            sum += i;

        System.out.println("모든 요소의 합은 " + sum + "입니다.");
    }
}
```
