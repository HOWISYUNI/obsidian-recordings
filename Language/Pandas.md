# 문법
## pandas.DataFrame.to_csv()
![[Pasted image 20220906190731.png]]
1. parameters
	- header=False : dataframe에서 필드 이름 라인 제거하여 저장
	- index=False : dataframe에서 행 번호 삭제하여 저장
# 특징
## dataframe iteration
| 속도순위 | method     | `iterrrows` 대비 속도 |
| -------- | ---------- | --------------------- |
| 1        | itertuples | x 8.1                 |
| 2        | at/iat     | x 4                   |
| 3        | loc/iloc   | x 2.5                 |
| 4        | iterrrows  | x 1                   |
![[Pasted image 20220901171933.png]]

![[Pasted image 20220901171958.png]]

![[Pasted image 20220901171910.png]]

![[Pasted image 20220901171925.png]]
