- 소수와 매우 크거나 작은 수를 표현할 때 사용
- 과학적 표기법 이용
	- ex) x.xxx * $10^{y}$
- 두 가지 표현
	- 단정밀도(32-bit)
	- 배정밀도(64-bit)

#### IEEE FP Format
- 호환성, 이식성을 위해 IEEE에서 STD를 정했음
	![[IMG_E2298E645623-1.jpeg]]
- S: 부호 비트(0: 양수, 1: 음수)
- Exponent: 지수부
	- 실제 지수값은 +Bias를 해줘야함
		- Single: Bias=127 | Double: Bias=1023
	- 지수부에 부호 비트를 없앰으로써 더 넓은 범위의 수를 표현할 수 있고, 대소 비교가 편해짐
	- 지수부 00000000, 11111111는 예약되어 있음
		- 00000000: 정확히 0 또는 비정규화된 수(0.xxx: 아주아주 작은 수)
		- 11111111: NaN(Not a Number) 또는 무한대
- 가수부 정규화(Normalize Significand)
	- 1.xxx 형태로 유지하여 최상위 비트를 항상 1이 될 수 있다고 가정함
		- 숨겨진(hidden) 비트라고 부름
	- 따라서 소수 부분만 저장하고, 맨 앞의 1은 암묵적으로 존재한다고 간주하여 1비트를 절약할 수 있음
	- $1.xxx * 2^e$ 형태로 정규화되기 때문에 가수부의 값은 항상 $1.0 <= 가수부 < 2.0$ 범위에 속함

#### Single Precision Range
- 가장 작은 수
	- Exponent: 00000001
		- 실제 Exponent: 1 - 127 = -126
	- Fraction: 000...00
		- 유효숫자 = 1.0
	- $\pm1.0 \times 2^{-126} \approx \pm1.2 \times 10^{-38}$
- 가장 큰 수
	- Exponent: 11111110
		- 실제 Exponent: 254 - 127 = 126
	- Fraction: 111..11
		- 유효숫자 $\approx$ 2.0
	- $\pm2.0 \times 2^{+127} \approx \pm3.4 \times 10^{+38}$
- 6자리가 유효 자릿수
#### Double Precision Range
- 가장 작은 수
	- Exponent: 00000000001
		- 실제 Exponent: 1 - 1023 = -1022
	- Fraction: 000...00
		- 유효숫자 = 1.0
	- $\pm1.0 \times 2^{-1022} \approx \pm1.2 \times 10^{-308}$
- 가장 큰 수
	- Exponent: 11111111110
		- 실제 Exponent: 2046 - 1023 = 1023
	- Fraction: 111..11
		- 유효숫자 $\approx$ 2.0
	- $\pm2.0 \times 2^{+1023} \approx \pm3.4 \times 10^{+308}$
- 16자리가 유효 자릿수


### 비트 자체로는 어떤 뜻도 내포하고 있지 않음
=> 어떻게 해석할 것인가에 따라 의미가 부여됨!