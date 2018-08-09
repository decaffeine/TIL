## Ethereum
- Solidity를 통해 Smart contract를 쉽고 간단하게? 만들 수 있다
- 합의 알고리즘은 PoW, 앞으로는 PoS로 변경하는 것을 검토 중
- 화폐 단위는 Ether ( > Szabo > Wei)
- Smart contract의 실행 환경은 EVM (Ethereum Virtual Machine) 위에서 동작하므로 JVM처럼 OS에 종속되지 않음
    - 최근에는 라즈베리 파이용 바이너리도 제공 (IoT 적용을 위해)
- Go 언어판이 가장 활발하게 개발되는 중

---

### 환경설정
도커로 띄울 때는

```
docker run --rm -it ubuntu:16.04 /bin/bash
```

- git을 설치
```
apt-get update
apt-get instal -y git-core
```

- go-ethereum 클론
```
git clone -b release/1.3.6 https://github.com/ethereum/go-ethereum.git
```

- go 환경 설치
```
apt-get install -y build-eseential libgmp3-dev golang
```

- geth 빌드
```
make -C go-ethereum geth
```

- geth 빌드 확인 & /usr/bin으로 카피
```
./go-ethereum/build/bin/geth version
cp go-ethereum/build/bin/geth /usr/bin
```

#### 테스트 네트워크 구축
로컬 환경에서 이용 가능한 옵션을 사용
--networkid [네트워크 식별자] --datadir [워크 파일이 저장될 디렉토리] 
--olympic (테스트 네트워크 사용) --console (콘솔 모드)

```
mkdir eth_data
geth --networkid "123" --datadir "eth_data" --olympic console
```

```
instance: Geth/v1.3.6-bf324bd2/linux/go1.6.2
 datadir: eth_data
coinbase: null
at block: 0 (Thu, 01 Jan 1970 00:00:00 UTC)
modules: admin:1.0 db:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 shh:1.0 txpool:1.0 web3:1.0
```

---

#### 써봅시다

- 계좌 생성
```
personal.newAccount("testuser1")
personal.newAccount("testuser2")
```

```
"0x4e688be5d25e83a4a4445e358c01c4e6f8c5fb9d"
"0xd269c36b87d5822f30fd9bc16dec1e5ca9dc101a"
```

```
eth.accounts
```
- 잔고 확인
```
eth.getBalance(eth.accounts[0])
eth.getBalance("0x4e688be5d25e83a4a4445e358c01c4e6f8c5fb9d")
```

- 블록 수 확인
```
eth.blockNumber
```
- 채굴
```
miner.start()
miner.stop()
```

```
I0809 06:03:09.620026    4977 backend.go:584] Automatic pregeneration of ethash DAG ON (ethash dir: /root/.ethash)
I0809 06:03:09.620324    4977 miner.go:119] Starting mining operation (CPU=4 TOT=5)
true
> I0809 06:03:09.620756    4977 backend.go:591] checking DAG (ethash dir: /root/.ethash)
I0809 06:03:09.621417    4977 worker.go:569] commit new work on block 1 with 0 txs & 0 uncles. Took 982.724µs
I0809 06:03:09.621644    4977 ethash.go:220] Generating DAG for epoch 0 (size 1073739904) (0000000000000000000000000000000000000000000000000000000000000000)
I0809 06:03:10.407067    4977 ethash.go:252] Generating DAG: 0%
I0809 06:03:15.851836    4977 ethash.go:252] Generating DAG: 1%
I0809 06:03:21.301430    4977 ethash.go:252] Generating DAG: 2%
...
I0809 06:12:22.272106    4977 worker.go:348] 🔨  Mined block (#50 / 54a794f8). Wait 5 blocks for confirmation
I0809 06:12:22.272640    4977 worker.go:569] commit new work on block 51 with 0 txs & 0 uncles. Took 448.166µs
```

채굴이 종료된 뒤 잔고를 확인해보면
```
eth.getBalance("eth.accounts[0]")
eth.getBalance("eth.accounts[1]")
```

```
75000000000000000000
0
```

- 송금   
testuser1에서 testuser2로 송금!  
```
eth.sendTransaction({from:'0x4e688be5d25e83a4a4445e358c01c4e6f8c5fb9d',to:'0xd269c36b87d5822f30fd9bc16dec1e5ca9dc101a',value:web3.toWei(1,"ether")})
```

testuser1의 초기 비밀번호는 계정명과 동일(testuser1).  

```
Please unlock account 4e688be5d25e83a4a4445e358c01c4e6f8c5fb9d.
Passphrase: 
Account is now unlocked for this session.
I0809 06:15:37.331922    4977 xeth.go:1028] Tx(0xc505dfbb2b7338af474b2af72df3eb543ad1ed4e8eb92938bb99c587cc5eaecf) to: 0xd269c36b87d5822f30fd9bc16dec1e5ca9dc101a
```

미확정 트랜잭션 확인
```
eth.pendingTransactions
```

```
[{
    data: "0x",
    from: "0x4e688be5d25e83a4a4445e358c01c4e6f8c5fb9d",
    gas: "90000",
    gasPrice: "20000000000",
    hash: "0xc505dfbb2b7338af474b2af72df3eb543ad1ed4e8eb92938bb99c587cc5eaecf",
    nonce: "0",
    to: "0xd269c36b87d5822f30fd9bc16dec1e5ca9dc101a",
    value: "1000000000000000000"
}]
```

트랜잭션 확정을 위해 채굴
```
miner.start()
miner.stop()
```

- 송금 여부 확인

  - 다시 미확정 트랜잭션을 확인해 보면 null이 나온다.
  - testuser2의 잔고 확인
```
eth.getBalance(eth.accounts[1])
```
```
1000000000000000000
```


#### Contract를 사용한 샘플 개발  

