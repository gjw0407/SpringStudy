# 스프링 핵심 원리 기본편 - 6강 Component Scan

Component Scan: Component (Service, Controller, Repository), Configuration으로 구성한 Bean들을 자동으로 검색하고, Bean으로 등록

- Excludefilters: 특정 Annotation은 추가를 안함
- Basepackages: 해당 Path 하위의 Bean들을 조사함 (보통 SpringBootApplication main 파일에 선언)

![Untitled](https://user-images.githubusercontent.com/61227459/182264730-23aadee0-44cf-4e07-95b5-a7bb37d5edda.png)

Controller, Service, Repository → Component

Configuration → Component

**Configuration v Component 계열 차이**

- Configuration: @Bean으로 등록된 Method를 처리하여 Bean Definition을 반환할 것을 암시함
- Component: component auto-scan으로 등록될 Candidate임을 암시함