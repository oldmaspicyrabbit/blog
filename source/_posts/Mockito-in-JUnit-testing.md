---
title: Mockito in JUnit testing
date: 2021-01-05 09:12:28
tags: 
- mockito
categories: 
- web 
keywords: 
- java
- mockito
excerpt: Mockito in java
--- 
# Mockito

Mockito provides two ways to meet your requirements: using Annotations or do it **programmatically**.



[toc]



http://nocoder.org/mockito-annotations/

https://www.baeldung.com/mockito-annotations

https://zhuanlan.zhihu.com/p/44239404

https://yanbin.blog/mockito-capture-method-paramters/



## Mock or Spy?

Mock stubs all the methods in the class, in other words, all the methods will return null when calling any method in this mocked class since it's all stub methods.

Spy spies on the class, it will invoke the real method when calling it, but you can stub the methods according to your needs. It means it more like stubs part of the classes. It's quite useful when you just want to make use of the existing logic in the real class.

Let's take a look at a example:

```java
public class Service {
    public String service() {
        System.out.print("invoked!");
        return "real service!";
    }
}
```



### Mock

All the methods will return NULL in the mock class.

```java
Service mock = Mockito.mock(Service.class);
System.out.println(mock.service());
```

now you will find it return **NULL**.

If you want to mock the result we can add a statement like this

```java
when(mock.service()).thenReturn("mock");
```

It outputs: **mock**



### Spy

```java
Service spy = Mockito.spy(new Service());
```

This line doesn't mock any method, so it will invoke the real method- the constructor of Service.Class. Which means it will return **invoked! real service!**

If we want to mock the result, we need to mock the spied object, it has two ways to mock it:

  * doReturn

    ```java
    doReturn("spy!").when(spy).service();
    ```

    it returns **spy!**

 * thenReturn

   ```java
   when(spy.service()).thenReturn("spy!");
   ```

   it returns **invoked!spy!**  

if you use `when` to mock the result, the method will be invoked really, only changed the return value!

if you use `doReturn` , it will stub the method, and return the value you want only.



## Mockito Annotations

### @MockitoJUnitRunner

This annotation is used to enable Mockito those annotations. There're three ways to achieve this goal.

The first option we have is to **annotate the JUnit test with a *MockitoJUnitRunner*** as in the following example:

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoAnnotationTest {
    ...
}
```

Alternatively, we can **enable Mockito annotations programmatically** as well, by invoking *MockitoAnnotations.initMocks()*:

```java
@Before
public void init() {
    MockitoAnnotations.initMocks(this);
}
```

Lastly, **we can use a *MockitoJUnit.rule()*** as shown below:

```java
public class MockitoInitWithMockitoJUnitRuleUnitTest {

    @Rule
    public MockitoRule initRule = MockitoJUnit.rule();

    ...
}
```

In this case, we must remember to make our rule *public*.

#### @Rule

@Rule is JUnit annotation, it's used to notify Mockito that instantiate the class which annotated by @Mock.

#### @SpringJUnit4ClassRunner 

You should know that @RunWith is exclusive for a test class. Either `@RunWith(MockitoJUnitRunner.class)` or `@RunWith(SpringJUnit4ClassRunner.class)`) , so if you want to use mockito and SpringJUnit4ClassRunner at the smae time, you have to call `MockitoAnnotations.initMocks(this)` programatically in code.

[If you try to simply unit test a class without the dependencies, there is no need for the `SpringJUnit4ClassRunner`. This runner is able to generate a complete Spring context with (mock) objects you can define in your (test) application context configuration. With this mechanism the `SpringJUnit4ClassRunner` is much slower than the regular `MockitoJUnitRunner`.

The `SpringJUnit4ClassRunner` is very powerful for integration test purposes.

I default start with the `MockitoJUnitRunner` and if I reach the limits of this runner, for instance because I need to mock constructors, static methods or private variables, I switch to `PowerMockJUnitRunner`. This is for me a last resort as it normally tells the code is ‘bad’ and not written to be tested. Other runners are normally not necessary for isolated unit tests.](https://stackoverflow.com/questions/31101693/using-mockitojunitrunner-class-instead-of-springjunit4classrunner-class).



### **@Mock**

We can use *@Mock* to create and inject mocked instances without having to call *Mockito.mock* manually.

In the following example – we'll create a mocked *ArrayList* with the manual way without using *@Mock* annotation:

```java
@Test
public void whenNotUseMockAnnotation_thenCorrect() {
    List mockList = Mockito.mock(ArrayList.class);
    
    mockList.add("one");
    Mockito.verify(mockList).add("one");
    assertEquals(0, mockList.size());

    Mockito.when(mockList.size()).thenReturn(100);
    assertEquals(100, mockList.size());
}
```

And now we'll do the same but we'll inject the mock using the *@Mock* annotation:

```java
@Mock
List<String> mockedList;

@Test
public void whenUseMockAnnotation_thenMockIsInjected() {
    mockedList.add("one");
    Mockito.verify(mockedList).add("one");
    assertEquals(0, mockedList.size());

    Mockito.when(mockedList.size()).thenReturn(100);
    assertEquals(100, mockedList.size());
}
```

Note how – in both examples, we're interacting with the mock and verifying some of these interactions – just to make sure that the mock is behaving correctly.



### @Spy

Now – let's see how to use *@Spy* annotation to spy on an existing instance.

In the following example – we create a spy of a *List* with the old way without using *@Spy* annotation:

```java
@Test
public void whenNotUseSpyAnnotation_thenCorrect() {
    List<String> spyList = Mockito.spy(new ArrayList<String>());
    
    spyList.add("one");
    spyList.add("two");

    Mockito.verify(spyList).add("one");
    Mockito.verify(spyList).add("two");

    assertEquals(2, spyList.size());

    Mockito.doReturn(100).when(spyList).size();
    assertEquals(100, spyList.size());
}
```

Let's now do the same – spy on the list – but do so using the *@Spy* annotation:

```java
@Spy
List<String> spiedList = new ArrayList<String>();

@Test
public void whenUseSpyAnnotation_thenSpyIsInjectedCorrectly() {
    spiedList.add("one");
    spiedList.add("two");

    Mockito.verify(spiedList).add("one");
    Mockito.verify(spiedList).add("two");

    assertEquals(2, spiedList.size());

    Mockito.doReturn(100).when(spiedList).size();
    assertEquals(100, spiedList.size());
}
```

Note how, as before – we're interacting with the spy here to make sure that it behaves correctly. In this example we:

- Used the **real** method *spiedList.add()* to add elements to the *spiedList*.
- **Stubbed** the method *spiedList.size()* to return *100* instead of *2* using *Mockito.doReturn()*.



### @Captor

@Captor is used to capture argument in the runtime.

We can use the *@Captor* annotation to create an *ArgumentCaptor* instance.

In the following example – we create an *ArgumentCaptor* with the old way without using *@Captor* annotation:

```java
@Test
public void whenNotUseCaptorAnnotation_thenCorrect() {
    List mockList = Mockito.mock(List.class);
    ArgumentCaptor<String> arg = ArgumentCaptor.forClass(String.class);

    mockList.add("one");
    Mockito.verify(mockList).add(arg.capture());

    assertEquals("one", arg.getValue());
}
```

Let's now **make use of *@Captor*** for the same purpose – to create an *ArgumentCaptor* instance:

```java
@Mock
List mockedList;

@Captor 
ArgumentCaptor argCaptor;

@Test
public void whenUseCaptorAnnotation_thenTheSam() {
    mockedList.add("one");
    Mockito.verify(mockedList).add(argCaptor.capture());

    assertEquals("one", argCaptor.getValue());
}
```

Notice how the test becomes simpler and more readable when we take out the configuration logic.



### @InjectMocks

Now – let's discuss how to use *@InjectMocks* annotation – to inject mock fields into the tested object automatically.

In the following example – we use *@InjectMocks* to inject the mock *wordMap* into the *MyDictionary* *dic*:

```java
@Mock
Map<String, String> wordMap;

@InjectMocks
MyDictionary dic = new MyDictionary();

@Test
public void whenUseInjectMocksAnnotation_thenCorrect() {
    Mockito.when(wordMap.get("aWord")).thenReturn("aMeaning");

    assertEquals("aMeaning", dic.getMeaning("aWord"));
}
```

And here is the class *MyDictionary*:

```java
public class MyDictionary {
    Map<String, String> wordMap;

    public MyDictionary() {
        wordMap = new HashMap<String, String>();
    }
    public void add(final String word, final String meaning) {
        wordMap.put(word, meaning);
    }
    public String getMeaning(final String word) {
        return wordMap.get(word);
    }
}
```

## 

### Usages:

####  Injecting a Mock into a Spy

 we might want to inject a mock into a spy:

```java
@Mock
Map<String, String> wordMap;

@Spy
MyDictionary spyDic = new MyDictionary();
```

**However, Mockito doesn't support injecting mocks into spies,** and the following test results in an exception:

```java
@Test 
public void whenUseInjectMocksAnnotation_thenCorrect() { 
    Mockito.when(wordMap.get("aWord")).thenReturn("aMeaning"); 

    assertEquals("aMeaning", spyDic.getMeaning("aWord")); 
}
```

If we want to use a mock with a spy, we can manually inject the mock through a constructor:

```java
MyDictionary(Map<String, String> wordMap) {
    this.wordMap = wordMap;
}
```

Instead of using the annotation, we can now create the spy manually:

```java
@Mock
Map<String, String> wordMap; 

MyDictionary spyDic;

@Before
public void init() {
    MockitoAnnotations.initMocks(this);
    spyDic = Mockito.spy(new MyDictionary(wordMap));
}
```

The test will now pass.





#### Running into NPE While Using Annotation

Often, we may **run into \*NullPointerException\*** when we try to actually use the instance annotated with *@Mock* or *@Spy*:

```java
public class MockitoAnnotationsUninitializedUnitTest {

    @Mock
    List<String> mockedList;

    @Test(expected = NullPointerException.class)
    public void whenMockitoAnnotationsUninitialized_thenNPEThrown() {
        Mockito.when(mockedList.size()).thenReturn(1);
    }
}
```

Most of the time, this happens simply because we forgot to properly enable Mockito annotations.

So, we have to keep in mind that each time we want to use Mockito annotations, we must take an extra step and initialize them as we already explained [earlier](https://www.baeldung.com/mockito-annotations#enable-annotations).