# Spring-Hibernate

/**************************************************************************************************************************/
package com.example.spring.app;

import java.math.BigDecimal;
import java.sql.Date;
import java.util.List;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;

import com.example.spring.config.AppConfig;
import com.example.spring.domain.Application;
import com.example.spring.domain.Bond;
import com.example.spring.domain.Driver;
import com.example.spring.model.Employee;
import com.example.spring.model.NewEmployee;
import com.example.spring.service.EmployeeService;
import com.example.spring.service.FileService;
import com.example.spring.service.NewEmployeeService;

public class AppMain {

	public static void main(String args[]) {
		AbstractApplicationContext context1 = new 
				AnnotationConfigApplicationContext(AppConfig.class);

		// Byname Autowiring
		Application application = (Application) context1.getBean("application");
		System.out.println("Application Details : " + application);
		
		Driver driver = (Driver) context1.getBean("driver");
        System.out.println("Driver Details : " + driver);
        
        Bond bond = (Bond) context1.getBean("bond");
        bond.showCar();
        
        FileService service1 = (FileService) context1.getBean("fileService");
        service1.readValues();
        
        EmployeeService eService = (EmployeeService) context1.getBean("employeeService");
        
        /*
         * Register employee using service
         */
        Employee employee3 = new Employee();
        employee3.setName("Danny Theys");
        eService.registerEmployee(employee3);
        
        context1.close(); 
        
        hibernateIntegrationCall();
        
	}
	
	public static void hibernateIntegrationCall(){
		AbstractApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
		 
        NewEmployeeService service = (NewEmployeeService) context.getBean("newEmployeeService");
 
        /*
         * Create Employee1
         */
        NewEmployee employee1 = new NewEmployee();
        employee1.setName("Han Yenn");
        employee1.setJoiningDate(new Date(2010, 10, 10));
        employee1.setSalary(new BigDecimal(10000));
        employee1.setSsn("ssn00000001");
 
        /*
         * Create Employee2
         */
        NewEmployee employee2 = new NewEmployee();
        employee2.setName("Dan Thomas");
        employee2.setJoiningDate(new Date(2012, 11, 11));
        employee2.setSalary(new BigDecimal(20000));
        employee2.setSsn("ssn00000002");
 
        /*
         * Persist both Employees
         */
        service.saveEmployee(employee1);
        service.saveEmployee(employee2);
 
        /*
         * Get all employees list from database
         */
        List<NewEmployee> employees = service.findAllEmployees();
        for (NewEmployee emp : employees) {
            System.out.println(emp);
        }
 
        /*
         * delete an employee
         */
        service.deleteEmployeeBySsn("ssn00000002");
 
        /*
         * update an employee
         */
 
        NewEmployee employee = service.findBySsn("ssn00000001");
        employee.setSalary(new BigDecimal(50000));
        service.updateEmployee(employee);
 
        /*
         * Get all employees list from database
         */
        List<NewEmployee> employeeList = service.findAllEmployees();
        for (NewEmployee emp : employeeList) {
            System.out.println(emp);
        }
 
        context.close();
    }
}
/**************************************************************************************************************************/
package com.example.spring.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;

/*
 * Spring configuration class are the ones annotated with @Configuration. 
 * These classes contains methods annotated with @Bean. 
 * These @Bean annotated methods generates beans managed by Spring container.
 * */

@Configuration
@ComponentScan("com.example.spring")
//@ComponentScan(basePackages = "com.example.spring")
@PropertySource(value = { "classpath:application.properties" })
public class AppConfig {
	/*
	 * PropertySourcesPlaceHolderConfigurer Bean only required for @Value("{}")
	 * annotations. Remove this bean if you are not using @Value annotations for
	 * injecting properties.
	 */
	@Bean
	public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
		return new PropertySourcesPlaceholderConfigurer();
	}
}

/*
 * Notice @ComponentScan which will make Spring auto detect the annotated beans
 * via scanning the specified package and wire them wherever needed
 * (using @Resource or @Autowired ).
 */

/* NOTE: Refer FileServiceImpl.java to understand ${...} implementation */
/*
 * @PropertySource(value = { “classpath:application.properties” }) annotation
 * makes the properties available from named property file[s] (referred by value
 * attribute) to Spring Environment. Environment interface provides getter
 * methods to read the individual property in application. Notice the
 * PropertySourcesPlaceholderConfigurer bean method. This bean is required only
 * for resolving ${…} placeholders in @Value annotations. In case you don’t use
 * ${…} placeholders, you can remove this bean altogether.
 */

/*
 * Below are commonly used Spring annotation which makes a bean auto-detectable:
 * 
 * @Repository - Used to mark a bean as DAO Component on persistence layer
 * 
 * @Service - Used to mark a bean as Service Component on business layer
 * 
 * @Controller - Used to mark a bean as Controller Component on Presentation
 * layer
 * 
 * @Configuration - Used to mark a bean as Configuration Component.
 * 
 * @Component - General purpose annotation, can be used as a replacement for
 * above annotations.
 */
/**************************************************************************************************************************/
package com.example.spring.config;

import java.util.Properties;

import javax.sql.DataSource;

import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.orm.hibernate4.HibernateTransactionManager;
import org.springframework.orm.hibernate4.LocalSessionFactoryBean;
import org.springframework.transaction.annotation.EnableTransactionManagement;

/*@Configuration indicates that this class contains one or more bean methods annotated with @Bean producing beans manageable by spring container. In our case, this class represent hibernate configuration.
@ComponentScan is equivalent to context:component-scan base-package="..." in xml, providing with where to look for spring managed beans/classes.
@EnableTransactionManagement is equivalent to Spring’s tx:* XML namespace, enabling Spring’s annotation-driven transaction management capability.
@PropertySource is used to declare a set of properties(defined in a properties file in application classpath) in Spring run-time Environment, providing flexibility to have different values in different application environments.
 * */
@Configuration
@EnableTransactionManagement
@ComponentScan({ "com.example.spring.config" })
@PropertySource(value = { "classpath:application.properties" })
public class HibernateConfig {

	@Autowired
	private Environment environment;

	/* IMPORTANT
	 * Method sessionFactory() is creating a LocalSessionFactoryBean, which
	 * exactly mirrors the XML based configuration : We need a dataSource and
	 * hibernate properties (same as hibernate.properties). Thanks
	 * to @PropertySource, we can externalize the real values in a .properties
	 * file, and use Spring’s Environment to fetch the value corresponding to an
	 * item. Once the SessionFactory is created, it will be injected into Bean
	 * method transactionManager which may eventually provide transaction
	 * support for the sessions created by this sessionFactory.
	 */
	@Bean
	public LocalSessionFactoryBean sessionFactory() {
		LocalSessionFactoryBean sessionFactory = new LocalSessionFactoryBean();
		sessionFactory.setDataSource(dataSource());
		sessionFactory.setPackagesToScan(new String[] { "com.example.spring.model" });
		sessionFactory.setHibernateProperties(hibernateProperties());
		return sessionFactory;
	} 

	
	@Bean
	public DataSource dataSource() {
		DriverManagerDataSource dataSource = new DriverManagerDataSource();
		dataSource.setDriverClassName(environment.getRequiredProperty("jdbc.driverClassName"));
		dataSource.setUrl(environment.getRequiredProperty("jdbc.url"));
		dataSource.setUsername(environment.getRequiredProperty("jdbc.username"));
		dataSource.setPassword(environment.getRequiredProperty("jdbc.password"));
		return dataSource;
	}

	private Properties hibernateProperties() {
		Properties properties = new Properties();
		properties.put("hibernate.dialect", environment.getRequiredProperty("hibernate.dialect"));
		properties.put("hibernate.show_sql", environment.getRequiredProperty("hibernate.show_sql"));
		properties.put("hibernate.format_sql", environment.getRequiredProperty("hibernate.format_sql"));
		properties.put("hibernate.hbm2ddl.auto", environment.getRequiredProperty("hibernate.hbm2ddl.auto"));
		return properties;
	}

	
	@Bean
	@Autowired
	public HibernateTransactionManager transactionManager(SessionFactory s) {
		HibernateTransactionManager txManager = new HibernateTransactionManager();
		txManager.setSessionFactory(s);
		return txManager;
	}
}
/**************************************************************************************************************************/
package com.example.spring.dao;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;

public abstract class AbstractDao {

	@Autowired
	//Autowired here from HibernateConfig.java file
	//This class serve as base class for database related operations.
	private SessionFactory sessionFactory;
	
	/*** IMPORTANT NOTE ***/
	//Notice above, that SessionFactory we have created earlier in HibernateConfig,
	//will be auto-wired here.This class serve as base class for database related operations.

	protected Session getSession() {
		return sessionFactory.getCurrentSession();
	}

	public void persist(Object entity) {
		getSession().persist(entity);
	}

	public void delete(Object entity) {
		getSession().delete(entity);
	}
}
/**************************************************************************************************************************/
package com.example.spring.dao;

import com.example.spring.model.Employee;

public interface EmployeeDao {
	 
    void saveInDatabase(Employee employee);
}

/**************************************************************************************************************************/
package com.example.spring.dao;

import org.springframework.stereotype.Repository;

import com.example.spring.model.Employee;

@Repository("employeeDao")
public class EmployeeDaoImpl implements EmployeeDao{
 
    public void saveInDatabase(Employee employee) {
 
        /*
         * Logic to save in DB goes here
         */
        System.out.println("Employee "+employee.getName()+" is registered for assessment on "+ employee.getAssessmentDate());
         
    }
 
}

/**************************************************************************************************************************/
package com.example.spring.dao;

import java.util.List;

import com.example.spring.model.Employee;
import com.example.spring.model.NewEmployee;

public interface NewEmployeeDao {

	void saveEmployee(NewEmployee employee);

	List<NewEmployee> findAllEmployees();

	void deleteEmployeeBySsn(String ssn);

	NewEmployee findBySsn(String ssn);

	void updateEmployee(NewEmployee employee);
}
/**************************************************************************************************************************/
package com.example.spring.dao;

import java.util.List;

import javax.transaction.Transactional;

import org.hibernate.Criteria;
import org.hibernate.Query;
import org.hibernate.criterion.Restrictions;
import org.springframework.stereotype.Repository;

import com.example.spring.model.Employee;
import com.example.spring.model.NewEmployee;

@Repository("newEmployeeDao")

public class NewEmployeeDaoImpl extends AbstractDao implements NewEmployeeDao {

	public void saveEmployee(NewEmployee employee) {
		persist(employee);
	}

	@SuppressWarnings("unchecked")
	public List<NewEmployee> findAllEmployees() {
		Criteria criteria = getSession().createCriteria(NewEmployee.class);
		return (List<NewEmployee>) criteria.list();
	}

	public void deleteEmployeeBySsn(String ssn) {
		Query query = getSession().createSQLQuery("delete from NewEmployee where ssn = :ssn");
		query.setString("ssn", ssn);
		query.executeUpdate();
	}

	public NewEmployee findBySsn(String ssn) {
		Criteria criteria = getSession().createCriteria(NewEmployee.class);
		criteria.add(Restrictions.eq("ssn", ssn));
		return (NewEmployee) criteria.uniqueResult();
	}

	public void updateEmployee(NewEmployee employee) {
		getSession().update(employee);
	}
}

/**************************************************************************************************************************/
package com.example.spring.domain;

import javax.annotation.Resource;
import org.springframework.stereotype.Component;

@Component("application")
public class Application {

	/*
	 * Standard @Resource annotation marks a resource that is needed by the application
	 * It is analogous to @Autowired in that both injects beans by type when no attribute
	 * provided. But with name attribute,
	 * @Resource allows you to inject a bean by it’s name, which @Autowired does not.
	 * */
	@Resource(name = "applicationUser")
	private ApplicationUser user;
	
	/*
	 * In above code, Application’s user property is annotated with 
	 * @Resource(name=”applicationUser”). 
	 * In this case, a bean with name ‘applicationUser’ found in applicationContext 
	 * will be injected here.
	 * */

	@Override
	public String toString() {
		return "Application [user=" + user + "]";
	}
}

/**************************************************************************************************************************/
package com.example.spring.domain;

import org.springframework.stereotype.Component;

@Component("applicationUser")
public class ApplicationUser {
 
    private String name = "defaultName";
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    @Override
    public String toString() {
        return "ApplicationUser [name=" + name + "]";
    }
}

/**************************************************************************************************************************/
package com.example.spring.domain;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
public class Bond {

	@Autowired 	//here 'Car' will be injected or autowired
	/*
	 * Without 'Qualifier' , Spring was not able to decide which bean
	 * (Ferari or Mustang as both implements Car) to choose for auto-wiring ,
	 * it throws this exception
	 * */
	@Qualifier("Mustang")
	private Car car;

	public void showCar() {
		car.getCarName();
	}
}
/**************************************************************************************************************************/
package com.example.spring.domain;

public interface Car {
	 
    public void getCarName();
 
}

/**************************************************************************************************************************/
package com.example.spring.domain;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
 
@Component("driver")
public class Driver {
	
	//@Autowired   : autowired on field
    private License license;
    
    //@Autowired   : autowired on constructor
    /*public Driver(License license){
        this.license = license;
    }*/
    
    @Autowired //autowired on setter method
    public void setLicense(License license) {
        this.license = license;
    }
 
    @Override
    public String toString() {
        return "Driver [license=" + license + "]";
    }
    //getter
}

/**************************************************************************************************************************/
package com.example.spring.domain;

import org.springframework.stereotype.Component;

@Component("Ferrari")
public class Ferrari implements Car{
 
    public void getCarName() {
        System.out.println("This is Ferari");
    }
 
}

/**************************************************************************************************************************/
package com.example.spring.domain;

import org.springframework.stereotype.Component;

@Component
public class License {
 
    private String number="123456ABC";
 
    @Override
    public String toString() {
        return "License [number=" + number + "]";
    }
    //setters, getters
}

/**************************************************************************************************************************/
package com.example.spring.domain;

import org.springframework.stereotype.Component;

@Component("Mustang")
public class Mustang implements Car{
 
    public void getCarName() {
        System.out.println("This is Mustang");
    }
 
}

/**************************************************************************************************************************/
package com.example.spring.model;

import java.time.LocalDate;

public class Employee {

	private int id;

	private String name;

	private String assessmentDate;

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getAssessmentDate() {
		return assessmentDate;
	}

	public void setAssessmentDate(String string) {
		this.assessmentDate = string;
	}

	@Override
	public String toString() {
		return "Employee [id=" + id + ", name=" + name + "]";
	}

}
/**************************************************************************************************************************/
package com.example.spring.model;

import java.math.BigDecimal;
import java.sql.Date;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
import org.hibernate.annotations.Type;

 
@Entity
@Table(name="NEWEMPLOYEE")
public class NewEmployee {
 
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
 
    @Column(name = "NAME", nullable = false)
    private String name;
 
    @Column(name = "JOINING_DATE", nullable = false)
    @Type(type="java.sql.Date")
    private Date joiningDate;
 
    @Column(name = "SALARY", nullable = false)
    private BigDecimal salary;
     
    @Column(name = "SSN", unique=true, nullable = false)
    private String ssn;
 
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        this.id = id;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public Date getJoiningDate() {
        return joiningDate;
    }
 
    public void setJoiningDate(Date joiningDate) {
        this.joiningDate = joiningDate;
    }
 
    public BigDecimal getSalary() {
        return salary;
    }
 
    public void setSalary(BigDecimal salary) {
        this.salary = salary;
    }
 
    public String getSsn() {
        return ssn;
    }
 
    public void setSsn(String ssn) {
        this.ssn = ssn;
    }
 
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + id;
        result = prime * result + ((ssn == null) ? 0 : ssn.hashCode());
        return result;
    }
 
    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (!(obj instanceof NewEmployee))
            return false;
        NewEmployee other = (NewEmployee) obj;
        if (id != other.id)
            return false;
        if (ssn == null) {
            if (other.ssn != null)
                return false;
        } else if (!ssn.equals(other.ssn))
            return false;
        return true;
    }
 
    @Override
    public String toString() {
        return "Employee [id=" + id + ", name=" + name + ", joiningDate="
                + joiningDate + ", salary=" + salary + ", ssn=" + ssn + "]";
    }    
}
/**************************************************************************************************************************/
package com.example.spring.service;

import java.time.LocalDate;

public interface DateService {
 
    String getNextAssessmentDate();
}

/**************************************************************************************************************************/
package com.example.spring.service;

import java.util.Date;

import org.springframework.format.datetime.joda.LocalDateTimeParser;
import org.springframework.stereotype.Service;

@Service("dateService")
public class DateServiceImpl implements DateService {

	public String getNextAssessmentDate() {
		return new Date() + "";
	}

}

/**************************************************************************************************************************/
package com.example.spring.service;

import com.example.spring.model.Employee;

public interface EmployeeService {
	void registerEmployee(Employee employee);
}

/**************************************************************************************************************************/
package com.example.spring.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.example.spring.dao.EmployeeDao;
import com.example.spring.model.Employee;

@Service("employeeService")
public class EmployeeServiceImpl implements EmployeeService {
	@Autowired
	private DateService dateService;

	@Autowired
	private EmployeeDao employeeDao;

	public void registerEmployee(Employee employee) {
		employee.setAssessmentDate(dateService.getNextAssessmentDate());
		employeeDao.saveInDatabase(employee);
	}
}

/*
 * EmployeeService is our main service class.Notice that we have injected both
 * DateService and EmployeeDao in this.
 * 
 * @Autowired on dateService property marks the DateService to be auto-wired by
 * Spring’s dependency injection with the appropriate bean in Spring context. In
 * our case, we have already declared a DateService bean using @Service, so that
 * bean will be injected here. Similarly, employeeDao will be injected by
 * EmployeeDao annotated with @Repository.
 */

/**************************************************************************************************************************/
package com.example.spring.service;

public interface FileService {
	 
    void readValues();
}

/**************************************************************************************************************************/
package com.example.spring.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Service;

@Service("fileService")
public class FileServiceImpl implements FileService {

	//Default 'Value' template
	// @value("${key:default") 
	// private String var;
	
	@Value("${sourceLocation:c:/temp/input}")
	private String source;

	@Value("${destinationLocation:c:/temp/output}") 
	//observe the output, 'destinationLocation' has not been provided in application.properties file
	//hence, the default value '/temp/output' is being assigned as part of annotation
	private String destination;

	@Autowired
	private Environment environment;

	@Override
	public void readValues() {
		// TODO Auto-generated method stub
		System.out
				.println("Getting property via Spring Environment :" + environment.getProperty("jdbc.driverClassName"));

		System.out.println("Source Location : " + source);
		System.out.println("Destination Location : " + destination);

	}
}
/******** IMPORTANT *********/
/*
 * First point to notice is Environment got auto-wired by Spring. Thanks
 * to @PropertySoruce annotation , this Environment will get access to all the
 * properties declared in specified .properties file. You can get the value of
 * specific property using getProperty method. Several methods are defined in
 * Environment interface.
 * 
 * Other interesting point here is @Value annotation. Format of value annotation
 * is
 * 
 * @value("${key:default") 
 * private String var; 
 * 
 * Above declaration instruct Spring
 * to find a property with key named ‘key’ (from .properties file e.g.) and
 * assign it’s value to variable var.In case property ‘key’ not found, assign
 * value ‘default’ to variable var.
 * 
 * Note that above ${…} placeholder will only be resolved when we have
 * registered PropertySourcesPlaceholderConfigurer bean (which we have already
 * done above) else the @Value annotation will always assign default values to
 * variable var.
 */

/**************************************************************************************************************************/
package com.example.spring.service;

import java.util.List;

import com.example.spring.model.Employee;
import com.example.spring.model.NewEmployee;
 
public interface NewEmployeeService {
 
    void saveEmployee(NewEmployee employee);
 
    List<NewEmployee> findAllEmployees();
 
    void deleteEmployeeBySsn(String ssn);
 
    NewEmployee findBySsn(String ssn);
 
    void updateEmployee(NewEmployee employee);
}

/**************************************************************************************************************************/
package com.example.spring.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.spring.dao.NewEmployeeDao;
import com.example.spring.model.NewEmployee;

/****IMPORTANT NOTE****/
/*
 * Most interesting part above is @Transactional which starts a transaction
 * on each method start, and commits it on each method exit 
 * ( or rollback if method was failed due to an error). 
 * Note that since the transaction are on method scope, 
 * and inside method we are using DAO, DAO method will be executed within same transaction.
 * */
/**** IMPORTANT NOTE ****/

@Service("newEmployeeService")
@Transactional
public class NewEmployeeServiceImpl implements NewEmployeeService {

	@Autowired
	private NewEmployeeDao dao;

	/*** IMPORTANT NOTE */
	/*
	 * what is the mechanism to inject NewEmployeeDao dependency?
	 * */
	/*
	 * It's more towards Spring promoting programming to interfaces.
	 * NewEmployeeDaoImpl Bean is an implementation of NewEmployeeDao which
	 * would eventually be injected wherever asked. Eventually if you more than
	 * one implementation of that interface, then while injecting, you would
	 * need to specify which one Spring should be injecting using @Qualifier
	 */

	public void saveEmployee(NewEmployee employee) {
		dao.saveEmployee(employee);
	}

	public List<NewEmployee> findAllEmployees() {
		return dao.findAllEmployees();
	}

	public void deleteEmployeeBySsn(String ssn) {
		dao.deleteEmployeeBySsn(ssn);
	}

	public NewEmployee findBySsn(String ssn) {
		return dao.findBySsn(ssn);
	}

	public void updateEmployee(NewEmployee employee) {
		dao.updateEmployee(employee);
	}
}

/**************************************************************************************************************************/
jdbc.driverClassName = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/mysql
jdbc.username = root
jdbc.password = 
hibernate.dialect = org.hibernate.dialect.MySQLDialect
hibernate.show_sql = false
hibernate.format_sql = false
sourceLocation = /dev/input
hibernate.hbm2ddl.auto = create
/**************************************************************************************************************************/
/**************************************************************************************************************************/
/**************************************************************************************************************************/
/**************************************************************************************************************************/
