---
title: "자료구조와 함께 배우는 알고리즘 입문 (2) 배열, 난수, 역순 정렬, 두 배열 비교, 기수 변환, 소수의 나열"
date: 2020-06-02 23:55:00 +09:00
categories: programming
tags: [algorithm, datastructure, programming, book, IT]
---

책 <자료구조와 함께 배우는 알고리즘 입문>의 내용을 정리한 것입니다.

# 자료구조

자료구조란, 데이터 단위와 데이터 자체 사의 물리적 또는 논리적인 관계이다. 쉽게 말해 자료를 효율적으로 이용할 수 있도록 컴퓨터에 저장하는 방법을 말한다.

# 배열

`a = new int[5];`

위 선언의 의미는 int 형의 배열 본체를 새성하고 그것을 변수 a가 참조하도록 설정한다는 것이다. 배열의 구성 요소는 자동으로 0으로 초기화되는 규칙이 있다. 이 점은 보통 변수와 다르므로 꼭 기억해둬야 한다. 자료형에 따라 초깃값이 다른데 대부분 0이고, boolean의 경우 false, 참조형의 경우 공백을 참조하거나 null이 초깃값이 된다.

추가로, 메서드 안에서 선언한 지역 변수는 초깃값으로 초기화되지 않는다. 즉, 변수를 만들어도 초기화는 수행되지 않는다. 즉, 초기화되지 않은 변수로부터 값을 꺼낼 수는 없다.

## 배열의 복제

`배열이름.clone()`

clone 메서드를 호출하여 배열을 복제할 수 있다.

## 배열에서 최댓값 구하기

배열에서 최댓값 구하기의 원리는 다음과 같다. 최댓값을 저장할 변수(보통 max)를 만든다. 배열의 첫번째 값을 max에 저장한다. 배열의 두번째 요소의 값을 max의 값과 비교한다. max의 값이 더 작으면 배열의 두번째 요소의 값을 max에 저장하고 max의 값이 더 크면 배열의 세번째 요소의 값과 비교한다. 이렇게 배열의 마지막 요소의 값까지 max와 비교하면 최댓값을 구할 수 있다.

실제로 코드로 구현하기 위해서는, 배열과 최댓값 변수가 선언-초기화되어 있고, 배열 첫번째 요소부터 마지막 요소까지 조건문을 이용해 서로 다른 배열의 요소와 최댓값 변수의 값을 비교하는 조건문을 반복하도록 프로그램을 작성해야 한다.

# 난수

java.util에 패키지에 속한 Random 클래스는 Java가 제공하는 아주 큰 클래스 라이브러리입니다. Random 클래스의 인스턴스는 일련의 의사 난수를 생성합니다. 난수는 무(Nothing)에서 생성되는 것이 아니라 seed(씨앗)이라는 수의 값을 바탕으로 여러 연산을 수행하여 얻습니다.

# 역순 정렬

첫번째 값과 마지막 값을 바꾸고, 두번째 값을 (마지막-1)번째 값과 바꾸는 것을 반복하면 된다. 값을 교환할 때는 같은 자료형을 가진 변수를 하나 더 사용한다.

# 두 배열 비교

먼저 두 배열의 길이를 비교합니다. 그리고 두 배열의 첫번째 요소부터 마지막 요소까지 비교합니다.

# 기수 변환

정숫값을 임의의 기수로 변환하는 알고리즘.

```
package chap02;
import java.util.Scanner;
// 입력 받은 10진수를 2진수~36진수로 기수 변환하여 나타냄

class CardConvRev {
    // 정숫값 x를 r진수로 변환하여 배열 d에 아랫자리부터 넣어두고 자릿수를 반환합니다.
    static int cardConvR(int x, int r, char[] d) {
        int digits = 0;                        // 변환 후의 자릿수
        String dchar = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";

        do {
            d[digits++] = dchar.charAt(x % r);    // r로 나눈 나머지를 저장
            x /= r;
        } while (x != 0);
        return digits;
    }

    public static void main(String[] args) {
        Scanner stdIn = new Scanner(System.in);
        int no;                            // 변환하는 정수
        int cd;                            // 기수
        int dno;                        // 변환 후의 자릿수
        int retry;                        // 다시 한 번?
        char[] cno = new char[32];        // 변환 후 각 자리의 숫자를 넣어두는 문자의 배열

        System.out.println("10진수를 기수 변환합니다.");
        do {
            do {
                System.out.print("변환하는 음이 아닌 정수：");
                no = stdIn.nextInt();
            } while (no < 0);

            do {
                System.out.print("어떤 진수로 변환할까요? (2~36)：");
                cd = stdIn.nextInt();
            } while (cd < 2 || cd > 36);

            dno = cardConvR(no, cd, cno);        // no를 cd진수로 변환

            System.out.print(cd + "진수로는 ");
            for (int i = dno - 1; i >= 0; i--)    // 윗자리부터 차례로 나타냄
                System.out.print(cno[i]);
            System.out.println("입니다.");

            System.out.print("한 번 더 할까요? (1.예／0.아니오)：");
            retry = stdIn.nextInt();
        } while (retry == 1);
    }
}
```

# String 클래스

문자열을 나타내는 것은 java.lang 패키지에 소속된 String 클래스이다. 이 형은 (int, double 형 같이) 기본형이 아니다. String형 변수의 전형적인 선언은 다음과 같다.

```
String s = "ABC";
```

초기자의 "ABC"는 문자열 리터럴이다. 문자열 리터럴은 단순히 문자가 늘어서 잇는 것이 아니라 String형 인스턴스에 대한 참조이다. String 클래스는 문자열을 넣어두기 위한 문자 배열, 문자 수를 나타내는 필드 등을 갖고 있는 클래스이다. 따라서 변수 s와 그것이 참조하는 인스턴스는 이미지는 다음과 같다.

(그림: 변수 s가 "ABC"라고 적혀있는 박스를 참조한다. 그런데 그 박스 안에는 문자배열(`private final char[]`)안의 ABC, 문자 수(`private final int`)안의 3 값, 등 여러 작은 박스(인스턴스)가 들어가 있다.)

-   여기서 final 개념이 나왔는데 무슨 의미일까 궁금해서 찾아봤다. final은 상수, 메서드, 클래스 등을 선언하고 초기화한 뒤 변경하지 못하게 할 때 사용된다고 한다.  
    출처: [https://m.blog.naver.com/PostView.nhn?blogId=goddlaek&logNo=220889229659&proxyReferer=https:%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=goddlaek&logNo=220889229659&proxyReferer=https:%2F%2Fwww.google.com%2F)

`final char[]`형 배열의 요소에는 앞쪽부터 순서대로 문자 'A', 'B', 'C'가 들어 있다. 그리고 문자 수를 나타내는 final형 필드에는 3이 들어 있다. String 클래스에는 이 그림에 나타난 것 이외에도 필드, 생성사, 메서드가 있다.  
**즉, 예전에 String을 가지고 몇 번째 문자를 가져오거나, 문자 수를 세거나, 문자열들이 서로 같은지를 비교할 수 있었던 것은 모두 String이 단순한 문자열이 아니라, 여러 인스턴스와 메서드를 가진 클래스 String 형 변수였기 때문이었다!!!**

# 소수의 나열
