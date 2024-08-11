
ref: https://techblog.tabling.co.kr/%EA%B8%B0%EC%88%A0%EA%B3%B5%EC%9C%A0-%EC%A0%95%EC%A0%81-%EC%BD%94%EB%93%9C-%EB%B6%84%EC%84%9D-sonarqube-6b59fa9b6b85

## 레포지토리 생성

![[Pasted image 20240325180518.png]]

## docker image 준비

### 브랜치 분석 플러그인이 존재하는 Community Edition 이미지 


```
$ docker pull mc1arke/sonarqube-with-community-branch-plugin:lts
```

### 함께 연동하여 사용할 postgres 이미지


```sh
$ docker pull ademirjpfinflor/postgres:13
```


## docker 실행

```
```

```sh
docker-compose -f ./sonar.yml up
```

#### Trouble shooting

https://github.com/abiosoft/colima/issues/384

```log
max file descriptors [8192] for elasticsearch process is too low, increase to at least [65536]
max size virtual memory [52729364480] for user [elastic] is too low, increase to [unlimited]
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

```sh
$ colima start --edit
```

```
- provision: []
+ provision:
+   - mode: system
+     script: sysctl -w vm.max_map_count=262144
```

### 소나 실행 및 Github App 등록

![[Pasted image 20240325212206.png]]
![[Pasted image 20240325212244.png]]
![[Pasted image 20240325212501.png]]

![[Pasted image 20240325212736.png]]

![[Pasted image 20240326101808.png]]
![[Pasted image 20240326111920.png]]

![[Pasted image 20240326112050.png]]

![[Pasted image 20240326112159.png]]
![[Pasted image 20240326112439.png]]
![[Pasted image 20240326172722.png]]
### Github Actions 적용

#### 1. Code Analyze for Pull Request

```yml
name: Code Analyze Pull Request  
  
on:  
	pull_request:  
		types: [opened, reopened, synchronize]  
		branches:  
			- master  
			- develop  
env:  
	NODE_VERSION: 18.12.0  
  
jobs:  
	code_analysis:  
		runs-on: ubuntu-latest  
  
		steps:  
			- name: 'Checkout repository on branch: ${{ github.REF }}'  
			  uses: actions/checkout@v3  
			  with:  
				ref: ${{ github.HEAD_REF }}  
			- name: Retrieve entire repository history  
			  run: |  
				git fetch --prune --unshallow  
		  
			- name: Setup Node.js  
			  uses: actions/setup-node@v3  
			  with:  
				node-version: ${{ env.NODE_VERSION }}  
			- name: Cache node modules  
			  uses: actions/cache@v3  
			  id: npm-cache  
			  with:  
				path: '**/node_modules'  
				key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}  
				restore-keys: |  
				${{ runner.os }}-node-  
		  
			- name: Install Dependencies  
			  if: steps.npm-cache.outputs.cache-hit != 'true'  
			  run: npm install  
		  
			- name: Coverage Test  
			  continue-on-error: true  
			  run: npm run coverage  
		  
			- name: Run an analysis of the ${{ github.REF }} branch ${{ github.BASE_REF }} base  
			  uses: sonarsource/sonarqube-scan-action@master  
			  env:  
				SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}  
				SONAR_HOST_URL: ${{ secrets.SONAR_URL }}
```


#### 2. 특정 브랜치 push 



```yml
name: Code Analyze branch  
  
# then define on which event, here a push  
on:  
push:  
# and the target with some regex to match our specific branch names  
branches:  
- master  
- develop  
  
# We can now build our job  
jobs:  
code_analysis:  
  
runs-on: ubuntu-latest  
  
steps:  
- name: 'Checkout repository on branch: ${{ github.REF }}'  
uses: actions/checkout@v3  
with:  
ref: ${{ github.REF }}  
fetch-depth: 0  
  
- name: Setup Node.js  
uses: actions/setup-node@v3  
with:  
node-version: ${{ env.NODE_VERSION }}  
  
- name: Cache node modules  
uses: actions/cache@v3  
id: npm-cache  
with:  
path: '**/node_modules'  
key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}  
restore-keys: |  
${{ runner.os }}-node-  
  
- name: Install Dependencies  
if: steps.npm-cache.outputs.cache-hit != 'true'  
run: npm install  
  
- name: Coverage Test  
continue-on-error: true  
run: npm run coverage  
  
- name: 'Run an analysis of the ${{ github.REF }} branch'  
uses: sonarsource/sonarqube-scan-action@master  
env:  
SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}  
SONAR_HOST_URL: ${{ secrets.SONAR_URL }}
```
