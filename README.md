# Docker + CI/CD Practice

Kotlin/Spring Boot 헬스체크 API를 Docker로 컨테이너화하고, GitHub Actions로 AWS(ECR, EC2)에 자동 배포하는 CI/CD 파이프라인 실습 프로젝트입니다.

## Architecture

Local (Kotlin/Spring Boot) → git push → GitHub Actions (build JAR → build Docker image → push to ECR) → SSH deploy to EC2 (pull image → run container)

## Stack

- **Language/Framework**: Kotlin, Spring Boot 3.x
- **Containerization**: Docker
- **CI/CD**: GitHub Actions
- **Cloud**: AWS ECR, AWS EC2 (t3.micro)
- **IAM**: 전용 IAM 사용자(GitHub Actions용, ECR 권한만 최소 부여) + EC2 인스턴스 역할(ECR 읽기 전용)

## What this demonstrates

- Dockerfile 작성 및 이미지 빌드/실행
- Amazon ECR을 통한 이미지 저장 및 배포
- GitHub Actions 워크플로 작성 (.github/workflows/deploy.yml)
- GitHub Secrets를 통한 자격증명 관리 (AWS Access Key, EC2 SSH Key)
- 최소 권한 원칙에 따른 IAM 정책 분리

## Troubleshooting log

개발 및 배포 과정에서 직접 재현하고 해결한 이슈들:

| 이슈 | 원인 | 해결 |
|---|---|---|
| Gradle 빌드 실패 (Cannot find a Java installation) | JAVA_HOME 환경변수 미설정 | JDK 설치 경로 확인 후 setx JAVA_HOME 설정 |
| Docker Desktop 구동 실패 (Virtualization support not detected) | WSL2 관련 Windows 기능 비활성 | WSL2 활성화, wsl --update |
| ECR 로그인 거부 (AccessDeniedException) | EC2 IAM 역할에 ECR 권한 부재 | AmazonEC2ContainerRegistryReadOnly 정책 추가 |
| GitHub Actions 워크플로 미인식 | .github/workflows/가 저장소 루트가 아닌 하위 폴더에 위치 | 파일을 저장소 루트로 이동 |
| Build JAR 단계 실패 (Permission denied) | gradlew에 실행 권한 없이 커밋됨 (Windows 환경) | git update-index --chmod=+x |
| Build JAR 단계 실패 (Cannot find a Java installation, exit 126→1) | GitHub Actions 러너에 JDK 19 미설치 | 워크플로에 actions/setup-java@v4 단계 추가 |
| Deploy to EC2 실패 (i/o timeout on :22) | 보안 그룹 SSH 인바운드가 특정 IP로 제한되어 GitHub Actions 러너 IP 차단 | 22번 포트 인바운드 규칙 확장 |

## Local run

./gradlew bootRun
curl http://localhost:8080/health

## CI/CD

main 브랜치에 push하면 GitHub Actions가 자동으로 빌드, ECR 푸시, EC2 배포를 수행합니다.