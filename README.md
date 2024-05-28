
import com.github.tomakehurst.wiremock.WireMockServer
import static com.github.tomakehurst.wiremock.client.WireMock.*
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc
import org.springframework.test.web.servlet.MockMvc
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*
import org.springframework.test.web.servlet.result.MockMvcResultMatchers.*
import org.springframework.http.MediaType
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment
import org.springframework.boot.web.server.LocalServerPort
import spock.lang.Specification

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class DownstreamApiIntegrationTest extends Specification {

    @Autowired
    private MockMvc mockMvc

    @LocalServerPort
    private int port

    private static WireMockServer wireMockServer

    def setupSpec() {
        wireMockServer = new WireMockServer(8089)
        wireMockServer.start()
        configureFor("localhost", 8089)
        stubFor(get(urlEqualTo("/api/downstream"))
                .willReturn(aResponse()
                        .withHeader("Content-Type", "application/json")
                        .withBody('{"message": "Hello from the mocked API"}')))
    }

    def cleanupSpec() {
        wireMockServer.stop()
    }

    def "should retrieve data from downstream API"() {
        when:
        def result = mockMvc.perform(get("http://localhost:$port/api/your-endpoint")
                .contentType(MediaType.APPLICATION_JSON))

        then:
        result.andExpect(status().isOk())
              .andExpect(content().contentType(MediaType.APPLICATION_JSON))
              .andExpect(jsonPath('$.message').value("Hello from the mocked API"))
    }
}
