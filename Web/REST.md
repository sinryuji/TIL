"REpresentational State Transfer" 의 약자로, 자원을 이름(자원의 표현)으로 구분해 해당 자원의 상태(정보)를 주고 받는 모든 것을 의미합니다.


**1. 자원(Resource) - URI**

- 모든 자원에는 고유한 ID가 존재하고, 이 자원은 Server에 존재합니다.
- 자원을 구별하는 ID는 '/exgroups/:exgroup_id'와 같은 HTTP URI 입니다.
- Client는 URI를 이용해 자원을 지정하고 해당 자원의 상태(정보)에 대한 조작을 Server에 요청합니다.

**2. 행위(Verb) - Method**

- HTTP 프로토콜의 Method를 사용합니다.
- HTTP 프로토콜은 GET, POST, PUT, PATCH, DELETE의 Method를 제공합니다. ( CRUD )

**3. 표현 ( Representation of Resource )**

- Client와 Server가 데이터를 주고받는 형태로 JSON, XML, TEXT, RSS 등이 있습니다.
- JSON, XML을 통해 데이터를 주고 받는 것이 일반적입니다.

### REST의 특징

**1. Server-Client (서버-클라이언트 구조)**

- 자원이 있는 쪽이 Server, 자원을 요청하는 쪽이 Client가 됩니다.
    - REST Server는 API를 제공하고 비즈니스 로직 처리 및 저장을 책임지고,
    - Client는 사용자 인증이나 context( 세션, 로그인 정보 ) 등을 직접 관리하고 책임집니다.
    - 역할을 확실히 구분시킴으로써 서로 간의 의존성을 줄입니다.

**2. Stateless (무상태)**

- HTTP 프로토콜은 Stateless Protocol이므로 REST 역시 무상태성을 갖습니다.
- Client의 context를 Server에 저장하지 않습니다.
    - 즉, 세션과 쿠키와 같은 context 정보를 신경쓰지 않아도 되므로 구현이 단순해집니다.
- Server는 각각의 요청을 완전히 별개의 것으로 인식하고 처리합니다.
    - 각 API 서버는 Client의 요청만을 단순 처리합니다.
    - 즉, 이전 요청이 다음 요청의 처리에 연관되어서는 안됩니다. ( DB에 의해 바뀌는 것은 허용 )
    - Server의 처리 방식에 일관성을 부여하기 때문에 서비스의 자유도가 높아집니다.

**3. Cacheable (캐시 처리 기능)*
- 웹 표준 HTTP 프로토콜을 그대로 사용하므로 웹에서 사용하는 기존의 인프라를 그대로 활용할 수 있습니다.
    - 즉, HTTP가 가진 가장 강력한 특징 중 하나인 캐싱 기능을 적용할 수 있습니다.
    - HTTP 프로토콜 표준에서 사용하는 Last-Modified Tag 또는 E-Tag를 이용해 캐싱을 구현합니다.
- 대량의 요청을 효율적으로 처리할 수 있습니다.

**4. Layered System (계층 구조)**

- Client는 REST API Server만 호출합니다.
- REST Server는 다중 계층으로 구성될 수 있습니다.
    - 보안, 로드 밸런싱, 암호화 등을 위한 계층을 추가하여 구조를 변경할 수 있습니다.
    - Proxy, Gateway와 같은 네트워크 기반의 중간매체를 사용할 수 있습니다.
    - 하지만 Client는 Server와 직접 통신하는지, 중간 서버와 통신하는지는 알 수 없습니다.

**5. Uniform Interface (인터페이스 일관성)**

- URI로 지정한 Resource에 대한 요청을 통일되고, 한정적으로 수행하는 아키텍처 스타일을 의미합니다.
- HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하며, Loosely Coupling(느슨한 결함) 형태를 갖습니다. 
    - 즉, 특정 언어나 기술에 종속되지 않음


**6. Self-Descriptiveness (자체 표현)**
- 요청 메시지만 보고도 쉽게 이해할 수 있는 자체 표현 구조로 되어있습니다.
