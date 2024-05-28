// src/test/groovy/com/example/ControllerIntegrationTest.groovy
package com.example

import com.github.tomakehurst.wiremock.client.WireMock
import com.github.tomakehurst.wiremock.core.WireMockConfiguration
import com.github.tomakehurst.wiremock.WireMockServer
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.boot.test.web.client.TestRestTemplate
import org.springframework.boot.web.server.LocalServerPort
import org.springframework.http.ResponseEntity
import org.springframework.web.client.RestTemplate
import spock.lang.Specification
import spock.lang.Shared
import spock.lang.Stepwise

import static com.github.tomakehurst.wiremock.client.WireMock.*

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ControllerIntegrationTest extends Specification {

    @LocalServerPort
    private int port

    @Shared
    WireMockServer wireMockServer = new WireMockServer(WireMockConfiguration.wireMockConfig().dynamicPort())

    @Autowired
    private TestRestTemplate restTemplate

    def setupSpec() {
        wireMockServer.start()
        configureFor("localhost", wireMockServer.port())
        System.setProperty("downstream-service.url", "http://localhost:" + wireMockServer.port())
    }

    def cleanupSpec() {
        wireMockServer.stop()
    }

    def "should call downstream API and return expected result"() {
        given: "A stubbed downstream API"
        wireMockServer.stubFor(get(urlPathEqualTo("/api"))
                .willReturn(aResponse()
                        .withStatus(200)
                        .withHeader("Content-Type", "application/json")
                        .withBody('{"message": "Hello from downstream API"}')))

        when: "The controller endpoint is called"
        ResponseEntity<String> response = restTemplate.getForEntity("http://localhost:$port/your/controller/endpoint", String)

        then: "The response is as expected"
        response.statusCode.value() == 200
        response.body == '{"message": "Hello from downstream API"}'
    }
}
