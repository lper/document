本文主要讨论下Web开发中，准确而言，是PHP开发中的相关的设计模式及其应用。有经验的开发者肯定对于设计模式非常熟悉，但是本文主要是针对那些初级的开发者。首先我们要搞清楚到底什么是设计模式，设计模式并不是一种用来解释的模式，它们并不是像链表那样的常见的数据结构，也不是某种特殊的应用或者框架设计。事实上，设计模式的解释如下：

另一方面，设计模式提供了一种广泛的可重用的方式来解决我们日常编程中常常遇见的问题。设计模式并不一定就是一个类库或者第三方框架，它们更多的表现为一种思想并且广泛地应用在系统中。它们也表现为一种模式或者模板，可以在多个不同的场景下用于解决问题。设计模式可以用于加速开发，并且将很多大的想法或者设计以一种简单地方式实现。当然，虽然设计模式在开发中很有作用，但是千万要避免在不适当的场景误用它们。

目前常见的设计模式主要有23种，根据使用目标的不同可以分为以下三大类：


- 创建模式：用于创建对象从而将某个对象从实现中解耦合。
- 架构模式：用于在不同的对象之间构造大的对象结构。
- 行为模式：用于在不同的对象之间管理算法、关系以及职责。

#Creational Patterns

##Singleton(单例模式)
单例模式是最常见的模式之一，在Web应用的开发中，常常用于允许在运行时为某个特定的类创建一个可访问的实例。
    
	<?php
	/**
	 * Singleton class
	 */
	final class Product
	{

	    /**
	     * @var self
	     */
	    private static $instance;
	
	    /**
	     * @var mixed
	     */
	    public $mix;
	
	
	    /**
	     * Return self instance
	     *
	     * @return self
	     */
	    public static function getInstance() {
	        if (!(self::$instance instanceof self)) {
	            self::$instance = new self();
	        }
	        return self::$instance;
	    }
	
	    private function __construct() {
	    }
	
	    private function __clone() {
	    }
	}

	$firstProduct = Product::getInstance();
	$secondProduct = Product::getInstance();
	
	$firstProduct->mix = 'test';
	$secondProduct->mix = 'example';
	
	print_r($firstProduct->mix);
	// example
	print_r($secondProduct->mix);
	// example
在很多情况下，需要为系统中的多个类创建单例的构造方式，这样，可以建立一个通用的抽象父工厂方法：
    <?php

	abstract class FactoryAbstract {

	    protected static $instances = array();
	
	    public static function getInstance() {
	        $className = static::getClassName();
	        if (!(self::$instances[$className] instanceof $className)) {
	            self::$instances[$className] = new $className();
	        }
	        return self::$instances[$className];
	    }
	
	    public static function removeInstance() {
	        $className = static::getClassName();
	        if (array_key_exists($className, self::$instances)) {
	            unset(self::$instances[$className]);
	        }
	    }
	
	    final protected static function getClassName() {
	        return get_called_class();
	    }
	
	    protected function __construct() { }
	
	    final protected function __clone() { }
	}

	abstract class Factory extends FactoryAbstract {
	
	    final public static function getInstance() {
	        return parent::getInstance();
	    }
	
	    final public static function removeInstance() {
	        parent::removeInstance();
	    }
	}
	// using:
	
	class FirstProduct extends Factory {
	    public $a = [];
	}
	class SecondProduct extends FirstProduct {
	}
	
	FirstProduct::getInstance()->a[] = 1;
	SecondProduct::getInstance()->a[] = 2;
	FirstProduct::getInstance()->a[] = 3;
	SecondProduct::getInstance()->a[] = 4;
	
	print_r(FirstProduct::getInstance()->a);
	// array(1, 3)
	print_r(SecondProduct::getInstance()->a);
	// array(2, 4)
##Registry
注册台模式并不是很常见，它也不是一个典型的创建模式，只是为了利用静态方法更方便的存取数据。
    
	<?php
		/**
		* Registry class
		*/
	class Package {
	
	    protected static $data = array();
	
	    public static function set($key, $value) {
	        self::$data[$key] = $value;
	    }
	
	    public static function get($key) {
	        return isset(self::$data[$key]) ? self::$data[$key] : null;
	    }
	
	    final public static function removeObject($key) {
	        if (array_key_exists($key, self::$data)) {
	            unset(self::$data[$key]);
	        }
	    }
	}
	
	
	Package::set('name', 'Package name');
	
	print_r(Package::get('name'));
	// Package name
##Factory(工厂模式)
工厂模式是另一种非常常用的模式，正如其名字所示：确实是对象实例的生产工厂。某些意义上，工厂模式提供了通用的方法有助于我们去获取对象，而不需要关心其具体的内在的实现。
    
	<?php

	interface Factory {
	    public function getProduct();
	}
	
	interface Product {
	    public function getName();
	}
	
	class FirstFactory implements Factory {
	
	    public function getProduct() {
	        return new FirstProduct();
	    }
	}
	
	class SecondFactory implements Factory {
	
	    public function getProduct() {
	        return new SecondProduct();
	    }
	}
	
	class FirstProduct implements Product {
	
	    public function getName() {
	        return 'The first product';
	    }
	}
	
	class SecondProduct implements Product {
	
	    public function getName() {
	        return 'Second product';
	    }
	}
	
	$factory = new FirstFactory();
	$firstProduct = $factory->getProduct();
	$factory = new SecondFactory();
	$secondProduct = $factory->getProduct();
	
	print_r($firstProduct->getName());
	// The first product
	print_r($secondProduct->getName());
	// Second product
## abstractfactory(抽象工厂模式)

有些情况下我们需要根据不同的选择逻辑提供不同的构造工厂，而对于多个工厂而言需要一个统一的抽象工厂：

    <?php

	class Config {
	    public static $factory = 1;
	}
	
	interface Product {
	    public function getName();
	}
	
	abstract class AbstractFactory {
	
	    public static function getFactory() {
	        switch (Config::$factory) {
	            case 1:
	                return new FirstFactory();
	            case 2:
	                return new SecondFactory();
	        }
	        throw new Exception('Bad config');
	    }
	
	    abstract public function getProduct();
	}
	
	class FirstFactory extends AbstractFactory {
	    public function getProduct() {
	        return new FirstProduct();
	    }
	}
	class FirstProduct implements Product {
	    public function getName() {
	        return 'The product from the first factory';
	    }
	}
	
	class SecondFactory extends AbstractFactory {
	    public function getProduct() {
	        return new SecondProduct();
	    }
	}
	class SecondProduct implements Product {
	    public function getName() {
	        return 'The product from second factory';
	    }
	}
	
	$firstProduct = AbstractFactory::getFactory()->getProduct();
	Config::$factory = 2;
	$secondProduct = AbstractFactory::getFactory()->getProduct();
	
	print_r($firstProduct->getName());
	// The first product from the first factory
	print_r($secondProduct->getName());
	// Second product from second factory
##Object pool(对象池)

对象池可以用于构造并且存放一系列的对象并在需要时获取调用：

	<?php
	
	class Product {
	
	    protected $id;
	
	    public function __construct($id) {
	        $this->id = $id;
	    }
	
	    public function getId() {
	        return $this->id;
	    }
	}
	
	class Factory {
	
	    protected static $products = array();
	
	    public static function pushProduct(Product $product) {
	        self::$products[$product->getId()] = $product;
	    }
	
	    public static function getProduct($id) {
	        return isset(self::$products[$id]) ? self::$products[$id] : null;
	    }
	
	    public static function removeProduct($id) {
	        if (array_key_exists($id, self::$products)) {
	            unset(self::$products[$id]);
	        }
	    }
	}
	
	
	Factory::pushProduct(new Product('first'));
	Factory::pushProduct(new Product('second'));
	
	print_r(Factory::getProduct('first')->getId());
	// first
	print_r(Factory::getProduct('second')->getId());
	// second

##Lazy Initialization(延迟初始化)

对于某个变量的延迟初始化也是常常被用到的，对于一个类而言往往并不知道它的哪个功能会被用到，而部分功能往往是仅仅被需要使用一次。

    <?php

	interface Product {
	    public function getName();
	}
	
	class Factory {
	
	    protected $firstProduct;
	    protected $secondProduct;
	
	    public function getFirstProduct() {
	        if (!$this->firstProduct) {
	            $this->firstProduct = new FirstProduct();
	        }
	        return $this->firstProduct;
	    }
	
	    public function getSecondProduct() {
	        if (!$this->secondProduct) {
	            $this->secondProduct = new SecondProduct();
	        }
	        return $this->secondProduct;
	    }
	}
	
	class FirstProduct implements Product {
	    public function getName() {
	        return 'The first product';
	    }
	}
	
	class SecondProduct implements Product {
	    public function getName() {
	        return 'Second product';
	    }
	}
	
	
	$factory = new Factory();
	
	print_r($factory->getFirstProduct()->getName());
	// The first product
	print_r($factory->getSecondProduct()->getName());
	// Second product
	print_r($factory->getFirstProduct()->getName());
	// The first product

##Prototype(原型模式)

有些时候，部分对象需要被初始化多次。而特别是在如果初始化需要耗费大量时间与资源的时候进行预初始化并且存储下这些对象。
	
	<?php

	interface Product {
	}
	
	class Factory {
	
	    private $product;
	
	    public function __construct(Product $product) {
	        $this->product = $product;
	    }
	
	    public function getProduct() {
	        return clone $this->product;
	    }
	}
	
	class SomeProduct implements Product {
	    public $name;
	}
	
	
	$prototypeFactory = new Factory(new SomeProduct());
	
	$firstProduct = $prototypeFactory->getProduct();
	$firstProduct->name = 'The first product';
	
	$secondProduct = $prototypeFactory->getProduct();
	$secondProduct->name = 'Second product';
	
	print_r($firstProduct->name);
	// The first product
	print_r($secondProduct->name);
	// Second product


##Builder(构造者)

构造者模式主要在于创建一些复杂的对象：

	<?php
	
	class Product {
	
	    private $name;
	
	    public function setName($name) {
	        $this->name = $name;
	    }
	
	    public function getName() {
	        return $this->name;
	    }
	}
	
	abstract class Builder {
	
	    protected $product;
	
	    final public function getProduct() {
	        return $this->product;
	    }
	
	    public function buildProduct() {
	        $this->product = new Product();
	    }
	}
	
	class FirstBuilder extends Builder {
	
	    public function buildProduct() {
	        parent::buildProduct();
	        $this->product->setName('The product of the first builder');
	    }
	}
	
	class SecondBuilder extends Builder {
	
	    public function buildProduct() {
	        parent::buildProduct();
	        $this->product->setName('The product of second builder');
	    }
	}
	
	class Factory {
	
	    private $builder;
	
	    public function __construct(Builder $builder) {
	        $this->builder = $builder;
	        $this->builder->buildProduct();
	    }
	
	    public function getProduct() {
	        return $this->builder->getProduct();
	    }
	}
	
	$firstDirector = new Factory(new FirstBuilder());
	$secondDirector = new Factory(new SecondBuilder());
	
	print_r($firstDirector->getProduct()->getName());
	// The product of the first builder
	print_r($secondDirector->getProduct()->getName());
	// The product of second builder

#Structural Patterns

##Decorator(装饰器模式)

装饰器模式允许我们根据运行时不同的情景动态地为某个对象调用前后添加不同的行为动作。

	<?php
	class HtmlTemplate {
	    // any parent class methods
	}
	 
	class Template1 extends HtmlTemplate {
	    protected $_html;
	     
	    public function __construct() {
	        $this->_html = "<p>__text__</p>";
	    }
	     
	    public function set($html) {
	        $this->_html = $html;
	    }
	     
	    public function render() {
	        echo $this->_html;
	    }
	}
	 
	class Template2 extends HtmlTemplate {
	    protected $_element;
	     
	    public function __construct($s) {
	        $this->_element = $s;
	        $this->set("<h2>" . $this->_html . "</h2>");
	    }
	     
	    public function __call($name, $args) {
	        $this->_element->$name($args[0]);
	    }
	}
	 
	class Template3 extends HtmlTemplate {
	    protected $_element;
	     
	    public function __construct($s) {
	        $this->_element = $s;
	        $this->set("<u>" . $this->_html . "</u>");
	    }
	     
	    public function __call($name, $args) {
	        $this->_element->$name($args[0]);
	    }
	}

##Adapter(适配器模式)

这种模式允许使用不同的接口重构某个类，可以允许使用不同的调用方式进行调用：

	<?php

	class SimpleBook {
	
	    private $author;
	    private $title;
	
	    function __construct($author_in, $title_in) {
	        $this->author = $author_in;
	        $this->title  = $title_in;
	    }
	
	    function getAuthor() {
	        return $this->author;
	    }
	
	    function getTitle() {
	        return $this->title;
	    }
	}
	
	class BookAdapter {
	
	    private $book;
	
	    function __construct(SimpleBook $book_in) {
	        $this->book = $book_in;
	    }
	    function getAuthorAndTitle() {
	        return $this->book->getTitle().' by '.$this->book->getAuthor();
	    }
	}
	
	// Usage
	$book = new SimpleBook("Gamma, Helm, Johnson, and Vlissides", "Design Patterns");
	$bookAdapter = new BookAdapter($book);
	echo 'Author and Title: '.$bookAdapter->getAuthorAndTitle();
	
	function echo $line_in) {
	  echo $line_in."<br/>";
	}

#Behavioral Patterns

##Strategy(策略模式)

测试模式主要为了让客户类能够更好地使用某些算法而不需要知道其具体的实现。

	<?php

	interface OutputInterface {
	    public function load();
	}
	
	class SerializedArrayOutput implements OutputInterface {
	    public function load() {
	        return serialize($arrayOfData);
	    }
	}
	
	class JsonStringOutput implements OutputInterface {
	    public function load() {
	        return json_encode($arrayOfData);
	    }
	}
	
	class ArrayOutput implements OutputInterface {
	    public function load() {
	        return $arrayOfData;
	    }
	}

##Observer(观察者模式)

某个对象可以被设置为是可观察的，只要通过某种方式允许其他对象注册为观察者。每当被观察的对象改变时，会发送信息给观察者。

	<?php

	interface Observer {
	  function onChanged($sender, $args);
	}
	
	interface Observable {
	  function addObserver($observer);
	}
	
	class CustomerList implements Observable {
	  private $_observers = array();
	
	  public function addCustomer($name) {
	    foreach($this->_observers as $obs)
	      $obs->onChanged($this, $name);
	  }
	
	  public function addObserver($observer) {
	    $this->_observers []= $observer;
	  }
	}
	
	class CustomerListLogger implements Observer {
	  public function onChanged($sender, $args) {
	    echo( "'$args' Customer has been added to the list \n" );
	  }
	}
	
	$ul = new UserList();
	$ul->addObserver( new CustomerListLogger() );
	$ul->addCustomer( "Jack" );

##Chain of responsibility(责任链模式)

这种模式有另一种称呼：控制链模式。它主要由一系列对于某些命令的处理器构成，每个查询会在处理器构成的责任链中传递，在每个交汇点由处理器判断是否需要对它们进行响应与处理。每次的处理程序会在有处理器处理这些请求时暂停。

	<?php

	interface Command {
	    function onCommand($name, $args);
	}
	
	class CommandChain {
	    private $_commands = array();
	
	    public function addCommand($cmd) {
	        $this->_commands[]= $cmd;
	    }
	
	    public function runCommand($name, $args) {
	        foreach($this->_commands as $cmd) {
	            if ($cmd->onCommand($name, $args))
	                return;
	        }
	    }
	}
	
	class CustCommand implements Command {
	    public function onCommand($name, $args) {
	        if ($name != 'addCustomer')
	            return false;
	        echo("This is CustomerCommand handling 'addCustomer'\n");
	        return true;
	    }
	}
	
	class MailCommand implements Command {
	    public function onCommand($name, $args) {
	        if ($name != 'mail')
	            return false;
	        echo("This is MailCommand handling 'mail'\n");
	        return true;
	    }
	}
	
	$cc = new CommandChain();
	$cc->addCommand( new CustCommand());
	$cc->addCommand( new MailCommand());
	$cc->runCommand('addCustomer', null);
	$cc->runCommand('mail', null);