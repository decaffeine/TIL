- [블록체인 구조와 이론 : 예제로 배우는 핀테크 핵심 기술] 의 [비트코인] 챕터 참고

## bitcoin  
### 환경설정
ubuntu 16.04 환경 (superuser)

도커로 띄울 때는
```
docker run —-rm -it ubuntu:16.04 /bin/bash
```

#### wget과 git을 설치

```
apt-get update
apt-get -y install wget
apt-get -y install git-core
apt-get install software-properties-common
```

#### Bitcoin core 설치
```
mkdir src
cd src
git clone https://github.com/bitcoin/bitcoin.git
```

#### Bitcoin core에 필요한 라이브러리 모음 내려받기 (모두 apt-get install)
- gcc
- OpenSSL
- Boost
- libdb4.8
- 관련 라이브러리
- GUI 라이브러리

```
apt-get update
apt-get install automake pkg-config libevent-dev bsdmainutils
apt-get install libtool autotools-dev autoconf
apt-get install libssl-dev
apt-get install libboost-all-dev
add-apt-repository ppa:bitcoin/bitcoin
add-apt repository ppa:bitcoin/bitcoin
apt-get install software-properties-common
add-apt-repository ppa:bitcoin/bitcoin
apt-get update
apt-get install libdb4.8-dev
apt-get install libdb4.8++-dev
apt-get install libminiupnpc-dev
apt-get install libqrencode-dev -y
apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
```

#### 빌드
```
cd bitcoin
./autogen.sh
./configure
make
```

#### 설치
생각보다 시간이 좀 걸림

```
make install
```
---

### 테스트넷에서 가동

명령어로 실행 가능한 bitcoin-daemon(이후 bitcoind)을 사용
bitcoind에서는 옵션을 지정해 한정된 기기에서만 동작하는 로컬 비트코인 네트워크(테스트넷) 구축 가능

2개의 테스트 모드
- Testnet : 인터넷상에서 동작하는 테스트 네트워크
- Regtest : 로컬 PC 내에서의 테스트 네트워크

Regtest 모드를 사용하자.

```
bitcoind -regtest -daemon &
```

### Bitcoin Core 조작

- 블록 생성 및 블록 수 확인
```
bitcoin-cli -regtest generate 101
bitcoin0-cli -regtest getblockcount
```

- 계좌 생성
```
bitcoin-cli -regtest getnewaddress testuser1
```
```
2MseTZwpB8M8izEDa269j2gVtBUhWKc748G
```

- 잔고 확인
```
bitcoin-cli -regtest getbalance
bitcoin-cli -regtest getbalance testuser1 <- 이거 안 됨
```
- 송금 및 결과 확인
    - 송금
```
bitcoin-cli -regtest sendtoaddress (주소)
bitcoin-cli -regtest listunspent 0
```
0을 추가하면 미확정 트랜잭션도 함께 확인

	- 결과
```
[
  {
    "txid": "8a7461499d0023f22893c86c81065f16d28d6b6dc1bb28f6af0fec383e1add8f",
    "vout": 0,
    "address": "2N7KH8VjxWrKt5DSKK7iRKf4LnQWYtewQqz",
    "redeemScript": "001400fec3b7d9f583162c92ec92a4e0a0b0f2b78fc0",
    "scriptPubKey": "a9149a560c1b2f167a0cb0ab4b88e151db382c9990ca87",
    "amount": 39.99996240,
    "confirmations": 0,
    "spendable": true,
    "solvable": true,
    "safe": true
  },
  {
    "txid": "8a7461499d0023f22893c86c81065f16d28d6b6dc1bb28f6af0fec383e1add8f",
    "vout": 1,
    "address": "2MseTZwpB8M8izEDa269j2gVtBUhWKc748G",
    "label": "testuser1",
    "redeemScript": "0014900ba2aab8d703f256e15c2373e73f237fad00c0",
    "scriptPubKey": "a9140464e7487c3e2418ab80d018d2a5bfd86b684bdf87",
    "amount": 10.00000000,
    "confirmations": 0,
    "spendable": true,
    "solvable": true,
    "safe": true
  }
]
```

송금자의 계좌를 확인해 보면,
```
bitcoin-cli -regtest getbalance
```
```
49.99996240
```
수수료 차감된 결과임

### 채굴

미확정 트랜잭션을 확정하기 위해 채굴을 실행
```
bitcoin-cli -regtest generate 1
```
그리고
```
bitcoin-cli -regtest listunspent
bitcoin-cli -regtest getbalance testuser1 <- 이거 안 먹는데 어떻게 확인하지?
```
