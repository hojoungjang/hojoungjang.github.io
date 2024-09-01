---
layout: post
title:  "Public Key Cryptography"
date:   2024-08-29 16:07:00 +0900
categories: security
---
# 비대칭 암호화 (asymmetric cryptography)
비대칭 암호화에서는 public key 와 private key 가 존재한다.
Public key 는 말그대로 누구나 볼 수 있는 공유 자원이다. 반대로 private key 는 해당 key pair 를 소유하는 개인에게만 공개되어야 한다.

전체적인 아이디어는 하나의 키로 암호화를 할 경우 오직 암호화에 사용된 키의 짝이 되는 키로 복호화가 가능하다는 점에서 시작한다.

#### Confidentiality
만약 데이터를 외부로 부터 보호해야될 경우 public key로 공유하고 싶은 데이터를 암호화하고 private key 는 암호화된 데이터를 복호화 하는데 사용한다.

예를 들어 개발자 A 와 B 가 있다고 가정해보자. 이떄 A 와 B 모두 개인의 public-private key 쌍을 가지고 있다.
각각의 개발자들은 자신의 public key 는 외부와 공유를 하고 private key 는 자기 자신만이 가지고 있는다.

이제 A는 B에게 중요한 데이터를 보내고 싶다. 이 데이터는 오직 B에게만 보여져야 한다. 그러므로 A는 B의 public key 를 이용해서 데이터를 암호화해서
전송한다. A로부터 전달된 데이터를 B가 수신하고 자신의 private key 로 복호화해서 데이터를 확인한다.

#### Authenticity
반대의 경우도 존재한다. Private key 를 이용해 암호화를 진행하고 public key 로 복호화를 진행한다. 이떄 이루어지는 목표는 인증이다.

다시 개발자 A 와 B로 돌아가보자. 이번에도 A가 B에게 데이터를 보내는데 이번에는 데이터가 정말로 A 자기 자신이 보냈다는걸 B에게 확인시켜줘야 한다고 생각해보자.
그럴경우 A는 자기의 private key로 데이터를 암호화해서 B에게 보낸다. 그러면 암호화된 데이터를 전달 받은 B는 A의 public key로 복호화를 진행한다.

A의 private key는 오직 A만 가지고 있고 그것으로 암호화된 데이터는 A의 public key 로만 복호화가 가능하기 때문에 복호화가 성공한다면 B는 이 데이터가
정말로 A가 보냈다는걸 검증 할 수 있다.

Confidentiality 와 authenticity 둘 다 중요하기 때문에 보통은 위 두개의 방식을 섞어서 사용한다. 예를 들어 송신측은 1차적으로 자신의 private key 로
암호화하고 2차적으로 수신자의 public key로 암호화한다. 이때 1차적으로 일어나는 암호화는 사실상 암호화라고 부르기 보다는 서명을 (digital signature) 한다고 표현한다. 

# 장단점
## 대칭 암호화 (symmetric key cryptography)
1. 키 관리 [단점] - 각각의 호스트/개인이 여러개의 키를 관리해야한다. 만약 A 가 B 랑 C 와 주로 데이터를 주고 받는다고 생각하면 A 는 B 와 사용하는 키를 C랑도 사용하고 싶지 않을 것 이다. 그러므로 통신하는 상대방마다 유니크한 키를 요구한다. 개인이 관리해야하는 키의 개수가 어마무시 할 것 이다.

2. 키 분배 [단점] - 통신에 참여하는 호스트끼리의 키 공유과정에서 보안을 보장하기 힘들다. 하나의 약속된 키는 네트워크를 통해 공유해야 하는데 이 때 외부에서 통신을 가로채서 man-in-the-middle attack 또는 키 자체를 훔쳐 통신으로 엿볼 수 있다. 키가 외부로 노출될 경우 해당 키로 암호화한 모든 데이터는 노출이 될 수 있다.

3. 성능 [장점] - 하나의 키로 암호화와 복호화가 각각 한번 진행되기 때문이다. 위 예시처럼 보통 비대칭 암호화에서는 2번의 걸처서 암호화와 복호화가 각각 진행되기 때문에 연산과정이 비교적 더 오래 걸린다. 이 때 암호화하는 데이터가 크면 클 수록 성능 차이가 확연해 진다.

## 비대칭 암호화 (asymmetric key cryptography)
1. 키 관리 [장점] - 각각의 호스트/개인은 자신의 키만 잘 관리하면 된다. Public key 는 다른 상대방이 볼 수 있게 해놓고 private key 는 노출시키지 않으면 된다.

2. 키 분배 [장점] - 마찬가지로 public key만 잘 공유하면 끝이다.

3. 성능 [단점] - 여러번에 걸처 암호화와 복호화를 진행하기 때문에 큰 데이터를 대상으로는 상당히 비효율적일 수 있다.

그렇기 때문에 어떤 둘중 하나의 방법이 다른 하나를 대체하지는 않는다. 대표적인 예시로 SSL/TLS 에서 대칭과 비대칭 암호화를 섞어서 사용한다. 간단하게 요약하면 통신상에 있는 상대방과 통신을 위해 먼저 public, private key 를 사용해서 서로 상대방을 검증하고 대칭키에 대한 정보를 공유한다. 그 이후부터는 모든 메세지를 대칭키로 암호화해서 통신한다.

<!-- # RSA (Rivest, Shamir and Adleman)
RSA 는 여러 비대칭 암호화의 방식중 하나이다. 모듈러 연산과 소수를 사용한 수학적 접근방식으로 비대칭 암호화에 가능하게 하는 암호 키 쌍을 만들어내는 암호화 시스탬이다.

[rsa-eq](/assets/images/rsa_eq.png)

It uses `Fermat's little theorem` and `Euler's theorem`
It was shown by Rivest, Shamir, and Adleman that the above property holds for all M if n is a product of two
prime numbers.

If we choose a number `n` that is a product of two prime numbers `p` and `q`, we can choose an encryption value ,`e`,
such that it is 1 <= `e` < n which will be used to encrypt a message, `M`, like this `C = M ^ e mod n`. 

We can then choose a decryption value, `d`, such that it is modular multiplicative inverse of `e`. i.e. `(e * d) mod n = 1`.
Then, the resulting encrypted value `C` can be decrypted like this `M = (C ^ d) mod n`

안타깝지만 수학적 디테일은 꽤나 복잡하기 때문에 여기서는 생략하겠다.

mathematical attack
* trying to find prime factors of n
* semiprimes - a number that is multiple of two primes
* calculating number of factors that are relatively prime to n
* For semiprimes, computing the Euler totient function is equivalent to factoring

https://engineering.purdue.edu/kak/compsec/NewLectures/Lecture12.pdf -->
