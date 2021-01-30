### Spring RestDocs
- 의존성 추가
    ```
     <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-mockmvc</artifactId>
            <scope>test</scope>
    </dependency>
    ```
- 하는 역할 : 테스트를 돌렸을 떄 테스트를 한 것을 기반으로 api 문서 조각을 만들어준다.

- mvc 테스트에서 @AutoConfigureRestDocs 를 추가하고 .andDo(document("create-event")); 를 추가하여 사용할 수 있다. 이렇게 하면 create-event 폴더아래 api 문서 조각들이 생겨난다.

- 문서 형식을 알아보기 쉽게 만들기
    - @TestConfiguration : 테스트용 configuration을 만들 수 있다.
    ```
    @TestConfiguration
    public class RestDocsConfiguration {
        @Bean
        public RestDocsMockMvcConfigurationCustomizer restDocsMockMvcConfigurationCustomizer() {
            return new RestDocsMockMvcConfigurationCustomizer() {
                @Override
                public void customize(MockMvcRestDocumentationConfigurer configurer) {
                    configurer.operationPreprocessors()
                            .withRequestDefaults(prettyPrint())
                            .withResponseDefaults(prettyPrint());
                }
            };
        }
    }
    ```
    - 테스트에서 @Import(RestDocsConfiguration.class) 를 사용하여 빈을 사용할 수 있도록 한다.

- 문서 조각 만들기
    ```
    .andDo(document("create-event",
                            links(
                                    linkWithRel("self").description("link to self"),
                                    linkWithRel("query-events").description("link to query events"),
                                    linkWithRel("update-event").description("link to update event")
                            ),
                            requestHeaders(
                                    headerWithName(HttpHeaders.ACCEPT).description("accept header"),
                                    headerWithName(HttpHeaders.CONTENT_TYPE).description("content type header")
                            ),
                            requestFields(
                                    fieldWithPath("name").description("Name of new event"),
                                    ...모든 request 필드를 위와같이 작성한다.
                            ),
                            responseHeaders(
                                    headerWithName(HttpHeaders.LOCATION).description("Location header"),
                                    headerWithName(HttpHeaders.CONTENT_TYPE).description("Content type")
                            ),
                            responseFields(
                                    fieldWithPath("id").description("identifier of new event"),
                                    fieldWithPath("name").description("Name of new event"),
                                    ...모든 response 필드를 위와같이 작성한다.

                            )
    ```

- Rest Docs html 문서 만들기
    - main 폴더 밑에 asciidoc 폴더를 만들고 index.adoc 파일을 생성한다. html 템플릿 역할을 하는 문서를 작성한다.
    - 템플릿에 테스트 결과로 생성된 snippet들을 사용하여 하나의 html 문서를 만든다.
    - plugin을 추가해준다.
        - asciildoc 문서를 html 문서로 만들어준다.
        - 생성된 문서를 지정된 경로로 복사해준다.
    ```
    <plugin>
                <groupId>org.asciidoctor</groupId>
                <artifactId>asciidoctor-maven-plugin</artifactId>
                <version>1.5.3</version>
                <executions>
                    <execution>
                        <id>generate-docs</id>
                        <phase>prepare-package</phase>
                        <!-- 모든 asciidoc 문서를 html로 만들어준다. -->
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>html</backend>
                            <doctype>book</doctype>
                        </configuration>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.springframework.restdocs</groupId>
                        <artifactId>spring-restdocs-asciidoctor</artifactId>
                        <version>2.0.2.RELEASE</version>
                    </dependency>
                </dependencies>
            </plugin>
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.7</version>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <!--   목적지-->
                            <outputDirectory>
                                ${project.build.outputDirectory}/static/docs
                            </outputDirectory>
                            <!-- 옮기는 곳 -->
                            <resources>
                                <resource>
                                    <directory>
                                        ${project.build.directory}/generated-docs
                                    </directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
    ```

- profile 링크 추가
    - controller 코드에 추가해준다.
    - profile 링크는 해당 api의 문서 링크를 알려준다.
    ```
    eventResource.add(new Link("/docs/index.html#resources-events-create").withRel("profile"));
    ```