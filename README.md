## 인프라 구성도.

본 구성도는 쿠버네티스 환경에서 Front - Back - DB로 연결되는 3계층(3-tier) 구조를 나타냅니다.

가장 앞단에는 Ingress를 배치하여 애플리케이션 레벨에서 로드 밸런싱을 구성하고, 외부에서 들어오는 트래픽의 엔드포인트를 제한함으로써 외부 노출을 최소화했습니다. 또한, GKE 내에서는 각 애플리케이션 인스턴스를 서비스로 등록하고, 서비스 네임을 DNS에 등록하여 DNS를 통해 연결될 수 있도록 설정하였습니다. 이를 통해 클라이언트의 요청이 적절히 내부 서비스로 라우팅되고 로드밸런싱되도록 하였습니다.

프론트엔드 연결 단계에서 참조하는 FrontendConfig는 클라이언트의 HTTP 요청을 HTTPS로 리다이렉트하는 설정을 담고 있습니다. 이 구성 요소는 Ingress와 함께 사용되며, 보안이 강화된 HTTPS 프로토콜로만 클라이언트와의 통신이 이루어지도록 보장합니다.

루트 도메인으로 접속 시, 가장 먼저 연결되는 계층은 Front Tier입니다. 이 단계에서 Nginx와 React를 하나의 Docker 이미지로 빌드하여 배포를 간소화하고 리소스 최적화를 효과적으로 구현했습니다. 80포트로 요청이 들어오면 Nginx가 이를 받아 React 서비스로 전달하며, 이후 React Pod들과 연결됩니다. 사용자가 Front 단계에서 입력한 요청들은 API 요청 형태로 Ingress를 통해 spring-svc로 라우팅됩니다. 이때 Ingress는 Front에서 들어온 80포트의 요청을 8080포트로 매핑하여 전달합니다.

백엔드 연결 단계에서 참조하는 BackendConfig는 백엔드 서비스의 상태 확인(헬스 체크)과 세션 연결 방식(Session Affinity)을 관리하는 설정을 담고 있습니다. 헬스 체크를 통해 백엔드 서비스가 안정적으로 작동하는지 주기적으로 확인하고, 세션 어피니티 설정을 통해 쿠키를 사용하여 클라이언트의 요청이 동일한 백엔드 인스턴스로 라우팅되도록 하여 특정 세션에 대해 일관된 연결을 유지할 수 있도록 보장합니다.

DB와의 연결은 ConfigMap을 사용해 환경 변수를 통해 설정되며, 연결 정보는 DNS 이름으로 지정되었습니다. 이를 통해 Cloud SQL 인스턴스의 IP가 변경되더라도 지속적으로 연결이 가능하도록 구성하였습니다.
