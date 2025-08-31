12/09 확인 된 사항
242 서버에 최종 3개의 백엔드 프로젝트가 실행 중인 것으로 확인
- APTREE_AMANO_SERVER
- APTREE_Backend[TEST]
- APTREE_VISIT_SERVER

nssm 명령어를 통해 실행 중인 것으로 확인
```
sc qc MyJavaApp
```

nssm 설치 경로: 
D:\registerService\nssm64.exe

nssm 실행 방법 예시

```
D:\registerService\nssm64.exe edit APTREE_AMANO_SERVER
```

확인결과:
현재 모든 서비스는 bat 파일을 nssm에서 실행하고 있는 형태로 배포 중인 것으로 확인 완료

http://parking.aptree.co.kr:8280/
위의 루트로 서버 접근 가능!
FKbqxy8fn9jqdmof15gsu3w0onm