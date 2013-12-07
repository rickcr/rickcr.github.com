---
title: Jersey REST Web Services
layout: article
---
Jersey https://jersey.java.net/ is really awesome, however as a newbie to using it I found the documentation absolutely horrible.
A beginner is typically going to want to be handle typical CRUD actions using Java objects and it's extremely vague, at best, on
how to implement this.

What makes matters worse is when you start searching for examples, you end up getting a lot of Jersey 1.x examples and the API apparently
has changed quite a bit for 2.x (which this article covers.)

The full blown example you can access here https://github.com/rickcr/rick-jersey. Build with Maven using -DskipTests=true then deploy the war file to your
server of choice (I've tested with Tomcat 6.) Once the server war is deployed you can run the tests in the rick-jersey-client PersonTest class.

Below I've listed the two relevant portions of code, the server side jersey class and the client side code accessing the Jersey endpoints:

{% highlight java %}
import net.learntechnology.domain.Person;
import net.learntechnology.service.PersonService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.xml.bind.annotation.XmlElementWrapper;
import java.util.List;

@Path("/persons")
public class PersonWS {
	private final static Logger logger = LoggerFactory.getLogger(PersonWS.class);

	@Autowired
	private PersonService personService;

	@GET
	@Path("/{id}")
	@Produces({MediaType.APPLICATION_XML})
	public Person fetchPerson(@PathParam("id") Integer id) {
		return personService.fetchPerson(id);
	}

	@POST
	@Path("/add")
	@Consumes(MediaType.APPLICATION_XML)
	@Produces (MediaType.APPLICATION_XML)
	public Person addPerson(Person person) {
		personService.addPerson(person);
		return person;
	}

	@GET
	@Produces (MediaType.APPLICATION_XML)
	@XmlElementWrapper(name = "persons")
	public List<Person> fetchPersons() {
		List<Person> persons = personService.fetchAll();
		return persons;
	}

}
{% endhighlight %}


The client side code access the endpoints above:

{% highlight java %}
import net.learntechnology.domain.Person;
import org.junit.Assert;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.client.Entity;
import javax.ws.rs.core.GenericType;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import java.util.List;

public class PersonTest {
	private final static Logger logger = LoggerFactory.getLogger(PersonTest.class);

	@Test
	public void should_fetch_person_by_id() {
		Client client = ClientBuilder.newClient();
		Person person = client.target("http://localhost:8080/rick-jersey/rest/persons/2")
			.request(MediaType.APPLICATION_XML_TYPE).get(Person.class);
		logger.debug("Person: {}", person);
		Assert.assertEquals(new Integer(2), person.getId());
	}

	@Test
	public void should_add_person() {
		Person person = new Person(3, "Jeff Winger");
		Client client = ClientBuilder.newClient();
		Response response = client.target("http://localhost:8080/rick-jersey/rest/persons/add")
			.request(MediaType.APPLICATION_XML).post(Entity.entity(person, MediaType.APPLICATION_XML));
		person = response.readEntity(Person.class);

		//OR:
		//person = client.target("http://localhost:8080/rick-jersey/rest/person/add")
		//	.request(MediaType.APPLICATION_XML_TYPE).post(Entity.entity(person, MediaType.APPLICATION_XML), Person.class);

		logger.debug("response code: {}", response.getStatus());
		logger.debug("Person: {}", person);
		Assert.assertEquals(new Integer(3), person.getId());
	}

	@Test
	public void should_fetch_all() {
		Client client = ClientBuilder.newClient();
		List<Person> persons = client.target("http://localhost:8080/rick-jersey/rest/persons")
					.request(MediaType.APPLICATION_XML_TYPE).get(new GenericType<List<Person>>(){});
		logger.debug("persons = {}",persons);
		Assert.assertTrue(persons.size() > 0);
	}

}
{% endhighlight %}