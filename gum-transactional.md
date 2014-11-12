## Issue

 * Complex Transactions
 * Custom Propagation

~~
## Example

```groovy
	@Transactional(readOnly = true, propagation = Propagation.REQUIRES_NEW)
```

~~~~
## Code

```groovy
import org.codehaus.groovy.grails.orm.hibernate.cfg.DefaultGrailsDomainConfiguration
import org.hibernate.SessionFactory
import org.hibernate.cfg.Configuration
import org.hibernate.tool.hbm2ddl.SchemaExport

class NonTransactionalIntegrationSpec extends company.IntegrationSpec {

    @Shared
    private static Configuration _configuration

    static transactional = false

    def setupSpec() {
      //See Following Slide
    }

    def cleanupSpec() {
      //See Following Slide
    }

}
```
~~
## setupSpec()

```groovy
def setupSpec() {
    if (!_configuration) {
        // 1-time creation of the configuration
        Properties properties = new Properties()
        properties.setProperty 'hibernate.connection.driver_class', grailsApplication.config.dataSource.driverClassName
        properties.setProperty 'hibernate.connection.username', grailsApplication.config.dataSource.username
        properties.setProperty 'hibernate.connection.password', grailsApplication.config.dataSource.password
        properties.setProperty 'hibernate.connection.url', grailsApplication.config.dataSource.url
        properties.setProperty 'hibernate.dialect', 'org.hibernate.dialect.H2Dialect'

        _configuration = new DefaultGrailsDomainConfiguration(grailsApplication: grailsApplication, properties: properties)
    }
}
```
~~
## cleanupSpec()

```groovy
def cleanupSpec() {

    //After spec nuke and pave the test db
    new SchemaExport(_configuration).create(false, true)

    //Clear the sessions
    SessionFactory sf = grailsApplication.getMainContext().getBean('sessionFactory')
    sf.getCurrentSession().clear()
}
```
~~~~
## Inspiration

Burt's [An Alternative Approach for Grails Integration Tests](http://burtbeckwith.com/blog/?p=82)

We've updated that to work on Grails 2.2.x and 2.3.x with Spock.
