# 제목

## 표형식

  | 경로 | 설명 | Comment |
  | ------- | ---------------- | ---- |
  | .github/workflows | github action workflow 경로 |  |
  | actions | github action composite action 파일들이 있는 경로  |  |
  | airflow | airflow, 배치 관련 manifest 경로 | `airflow` namespace |  

<br>

## 들여쓰기(spacebar 2개씩)

- 디렉토리 / 파일명 공통 Rule

  - Repository 내 디렉토리 / 파일명은 기본적으로 `kebab case` 로 정의한다.  
    - (예) `helm-chart/argo-cd/values-dev.yaml`  
    - 특정 OS 에서 디렉토리 / 파일명에 대소문자 동시 사용시 case sensitive 가 적용되지 않는 issue 를 방지한다.
  - 단, document 전용 경로 내에서는 `한글` 로 디렉토리 / 파일명을 정의할 수 있다.
    - document 전용 경로 : `doc`

<br>

- Branch : 기본 `main` branch 만 사용하며, 별도의 branch 는 생성하지 않는다.  
  
  - Reference : Stop Using Branches for Deploying to Different GitOps Environments
    <https://link-url/>

- 상단 `Actios` Tab 을 클릭한다.

    <img src = "../_image/action_actions.png" alt="Actions" width="100%" height="100%" />
