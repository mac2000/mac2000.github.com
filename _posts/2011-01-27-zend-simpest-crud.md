---
layout: post
title: zend simplest crud

tags: [admin, create, crud, database, db, dbadapter, delete, edit, mysql, php, sql, update, zend]
---

Here is table:

```sql
CREATE TABLE customer (
    id int(11) NOT NULL AUTO_INCREMENT,
    first_name varchar(100) NOT NULL,
    last_name varchar(100) NOT NULL,
    age int(3) NOT NULL,
    email varchar(100) NOT NULL,
    active int(1) NOT NULL,
    PRIMARY KEY (id)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

Models:

```sh
zf create db-table Customer customer
zf create model CustomerMapper
zf create model Customer
```

Controller and its actions:

```sh
zf create controller Customer
zf create action list Customer
zf create action view Customer
zf create action create Customer
zf create action edit Customer
zf create action delete Customer
```

Form:

```sh
zf create form customerForm
```

Almost all code was copy pasted from http://framework.zend.com/manual/en/learning.quickstart.intro.html

**Application.ini config for mysql connection:**

```ini
resources.db.adapter = "PDO_MYSQL"
resources.db.params.host = "localhost"
resources.db.params.username = "root"
resources.db.params.password = ""
resources.db.params.dbname = "simplecrud_zf"
resources.db.isDefaultTableAdapter = true
```

**models/DbTable/Customer.php**

```php
<?php
//zf create db-table Customer customer
class Application_Model_DbTable_Customer extends Zend_Db_Table_Abstract
{
    protected $_name = 'customer';
}
```

**models/Customer.php**

```php
<?php

//zf create model Customer
class Application_Model_Customer
{
    /**
        * Customer ID
        *
        * @var int
        */
    protected $_id;
    /**
        * Customer first name
        *
        * @var string
        */
    protected $_firstName;
    /**
        * Customer last name
        *
        * @var string
        */
    protected $_lastName;
    /**
        * Customer email
        *
        * @var string
        */
    protected $_email;
    /**
        * Customer age
        *
        * @var int
        */
    protected $_age;
    /**
        * Is customer active
        *
        * @var bool
        */
    protected $_active;

    /**
        * Get customer ID
        *
        * @return int
        */
    public function getId()
    {
        return $this->_id;
    }

    /**
        * Set customer ID
        *
        * @param int $id
        * @return Application_Model_Customer
        */
    public function setId($id)
    {
        $this->_id = $id;
        return $this;
    }

    /**
        * Get customer first name
        *
        * @return string
        */
    public function getFirstName()
    {
        return $this->_firstName;
    }

    /**
        * Set customer first name
        *
        * @param string $firstName
        * @return Application_Model_Customer
        */
    public function setFirstName($firstName)
    {
        $this->_firstName = $firstName;
        return $this;
    }

    /**
        * Get customer last name
        *
        * @return string
        */
    public function getLastName()
    {
        return $this->_lastName;
    }

    /**
        * Set customer last name
        *
        * @param string $lastName
        * @return Application_Model_Customer
        */
    public function setLastName($lastName)
    {
        $this->_lastName = $lastName;
        return $this;
    }

    /**
        * Get customer email
        *
        * @return string
        */
    public function getEmail()
    {
        return $this->_email;
    }

    /**
        * Set customer email
        *
        * @param string $email
        * @return Application_Model_Customer
        */
    public function setEmail($email)
    {
        $this->_email = $email;
        return $this;
    }

    /**
        * Get customer age
        *
        * @return int
        */
    public function getAge()
    {
        return $this->_age;
    }

    /**
        * Set customer age
        *
        * @param int $age
        * @return Application_Model_Customer
        */
    public function setAge($age)
    {
        $this->_age = $age;
        return $this;
    }

    /**
        * Get customer active status
        *
        * @return bool
        */
    public function getActive()
    {
        return $this->_active;
    }

    /**
        * Set customer active status
        *
        * @param bool $active
        * @return Application_Model_Customer
        */
    public function setActive($active)
    {
        $this->_active = $active;
        return $this;
    }
}
```

**models/CustomerMapper.php**

```php
<?php

//zf create model CustomerMapper
class Application_Model_CustomerMapper
{
    /**
        * Customer table
        *
        * @var Application_Model_DbTable_Customer
        */
    protected $_dbTable;

    /**
        * Get customer table
        *
        * @return Application_Model_DbTable_Customer
        */
    public function getDbTable()
    {
        if (null === $this->_dbTable) {
            $this->setDbTable('Application_Model_DbTable_Customer');
        }

        return $this->_dbTable;
    }

    /**
        * Set customer table
        *
        * @param string, Application_Model_DbTable_Customer $dbTable
        * @return Application_Model_CustomerMapper
        */
    public function setDbTable($dbTable)
    {
        if (is_string($dbTable)) {
            $dbTable = new $dbTable();
        }

        if (!$dbTable instanceof Zend_Db_Table_Abstract) {
            throw new Exception('Invalid table data gateway provided');
        }

        $this->_dbTable = $dbTable;

        return $this;
    }

    public function save(Application_Model_Customer $customer)
    {
        $data = array(
            'id' => $customer->getId(),
            'first_name' => $customer->getFirstName(),
            'last_name' => $customer->getLastName(),
            'email' => $customer->getEmail(),
            'age' => $customer->getAge(),
            'active' => $customer->getActive(),
        );

        if (null === ($id = $customer->getId())) {
            unset($data['id']);
            $id = $this->getDbTable()->insert($data);
            $customer->setId($id);
        } else {
            $this->getDbTable()->update($data, array('id = ?' => $id));
        }
    }

    public function delete($id)
    {
        $row = $this->getDbTable()->find($id)->current();
        if ($row) {
            $row->delete();
            return true;
        } else {
            throw new Zend_Exception("Delete function failed; could not find row!");
        }
    }

    /**
        * Find customer by ID
        *
        * @param int $id
        * @return Application_Model_Customer
        */
    public function find($id)
    {
        $result = $this->getDbTable()->find($id);
        if (0 == count($result)) {
            return;
        }

        $row = $result->current();

        $entry = new Application_Model_Customer();
        $entry->setId($row->id)
                ->setFirstName($row->first_name)
                ->setLastName($row->last_name)
                ->setEmail($row->email)
                ->setAge($row->age)
                ->setActive($row->active);

        return $entry;
    }

    /**
        * Fetch all customers
        *
        * @return array
        */
    public function fetchAll()
    {
        $resultSet = $this->getDbTable()->fetchAll();
        $entries = array();
        foreach ($resultSet as $row) {
            $entry = new Application_Model_Customer();
            $entry->setId($row->id)
                    ->setFirstName($row->first_name)
                    ->setLastName($row->last_name)
                    ->setEmail($row->email)
                    ->setAge($row->age)
                    ->setActive($row->active);
            $entries[] = $entry;
        }

        return $entries;
    }

}
```

**Application/controllers/CustomerController.php**

```php
<?php

//zf create controller Customer
class CustomerController extends Zend_Controller_Action
{

    public function init()
    {
        /* Initialize action controller here */
    }

    public function indexAction()
    {
        $this->_redirect('/customer/list');
    }

    //zf create action list Customer
    public function listAction()
    {
        $customerMapper = new Application_Model_CustomerMapper();
        $this->view->entries = $customerMapper->fetchAll();
    }

    //zf create action view Customer
    public function viewAction()
    {
        $id = (int) $this->getRequest()->getParam('id');
        $customerMapper = new Application_Model_CustomerMapper();
        $this->view->entry = $customerMapper->find($id);
    }

    //zf create action create Customer
    public function createAction()
    {
        $form = new Application_Form_CustomerForm();
        $form->setAction('/customer/create');
        $form->setMethod('post');

        if ($this->getRequest()->isPost()) {
            if ($form->isValid($_POST)) {

                $customerMapper = new Application_Model_CustomerMapper();

                $customer = new Application_Model_Customer();
                $customer->setActive($form->getValue('active'))
                        ->setAge($form->getValue('age'))
                        ->setEmail($form->getValue('email'))
                        ->setFirstName($form->getValue('firstName'))
                        ->setLastName($form->getValue('lastName'));

                $customerMapper->save($customer);

                if ($customer->getId()) {
                    $this->_forward('list');
                }
            }
        }

        $this->view->form = $form;
    }

    //zf create action edit Customer
    public function editAction()
    {
        $form = new Application_Form_CustomerForm();
        $form->setAction('/customer/edit');
        $form->setMethod('post');

        $customerMapper = new Application_Model_CustomerMapper();

        if ($this->getRequest()->isPost()) {
            if ($form->isValid($_POST)) {
                $customer = new Application_Model_Customer();

                $customer->setId($form->getValue('id'))
                        ->setActive($form->getValue('active'))
                        ->setAge($form->getValue('age'))
                        ->setEmail($form->getValue('email'))
                        ->setFirstName($form->getValue('firstName'))
                        ->setLastName($form->getValue('lastName'));

                $customerMapper->save($customer);

                $this->_forward('list');
            }
        } else {
            $id = $this->getRequest()->getParam('id');
            $customer = $customerMapper->find($id);

            $values = array(
                'id' => $customer->getId(),
                'firstName' => $customer->getFirstName(),
                'lastName' => $customer->getLastName(),
                'email' => $customer->getEmail(),
                'age' => $customer->getAge(),
                'active' => $customer->getActive(),
            );

            $form->populate($values);
        }

        $this->view->form = $form;
    }

    //zf create action delete Customer
    public function deleteAction()
    {
        $id = $this->getRequest()->getParam('id');
        $customerMapper = new Application_Model_CustomerMapper();
        $customerMapper->delete($id);
        $this->_redirect('/customer/list');
    }

}
```

**Application/forms/CustomerForm.php**

```php
<?php

//zf create form customerForm
class Application_Form_CustomerForm extends Zend_Form
{

    public function init()
    {
        $id = $this->createElement('hidden', 'id');
        $this->addElement($id);

        $firstName = $this->createElement('text', 'firstName');
        $firstName->setLabel('First name:');
        $firstName->setRequired();
        $firstName->setAttrib('size', 30);
        $this->addElement($firstName);

        $lastName = $this->createElement('text', 'lastName');
        $lastName->setLabel('Last name:');
        $lastName->setRequired();
        $lastName->setAttrib('size', 30);
        $this->addElement($lastName);

        $email = $this->createElement('text', 'email');
        $email->setLabel('Email:');
        $email->setRequired();
        $email->addValidator(new Zend_Validate_EmailAddress());
        $email->addFilters(array(
            new Zend_Filter_StringTrim(),
            new Zend_Filter_StringToLower(),
        ));
        $email->setAttrib('size', 30);
        $this->addElement($email);

        $age = $this->createElement('text', 'age');
        $age->setLabel('Age:');
        $age->setRequired();
        $age->addValidator(new Zend_Validate_Int());
        $age->setAttrib('size', 30);
        $this->addElement($age);

        $active = $this->createElement('checkbox', 'active');
        $active->setLabel('Active:');
        $this->addElement($active);

        $this->addElement('submit', 'submit', array('label' => 'Submit'));
    }

}
```

**views/scripts/customer/list.phtml**

```php
<?php /* @var $this Zend_View */ ?>

<a href="/customer/create">create customer</a>
<table cellpadding="2" cellspacing="0" border="1">
    <caption>Customers list</caption>
    <tr>
        <th>id</th>
        <th>first name</th>
        <th>last name</th>
        <th>email</th>
        <th>age</th>
        <th>active</th>
        <th>info</th>
        <th>edit</th>
        <th>delete</th>
    </tr>
    <?php foreach ($this->entries as $entry): /* @var $entry Application_Model_Customer */ ?>
        <tr>
            <td><?php echo $this->escape($entry->getId()); ?></td>
            <td><?php echo $this->escape($entry->getFirstName()); ?></td>
            <td><?php echo $this->escape($entry->getLastName()); ?></td>
            <td><?php echo $this->escape($entry->getEmail()); ?></td>
            <td><?php echo $this->escape($entry->getAge()); ?></td>
            <td><?php echo $this->escape($entry->getActive()); ?></td>
            <td><a href="/customer/view/id/<?php echo $this->escape($entry->getId()); ?>">info</a></td>
            <td><a href="/customer/edit/id/<?php echo $this->escape($entry->getId()); ?>">edit</a></td>
            <td><a href="/customer/delete/id/<?php echo $this->escape($entry->getId()); ?>">delete</a></td>
        </tr>
    <?php endforeach; ?>
</table>
<a href="/customer/create">create customer</a>
```

**views/scripts/customer/view.phtml**

```php
<?php /* @var $this Zend_View */ ?>
<?php /* @var $entry Application_Model_Customer */ $entry = $this->entry; ?>

<a href="/customer/list">back to list</a>
<table cellpadding="2" cellspacing="0" border="1">
    <caption>Customer info</caption>

    <tr>
        <th>id</th>
        <td><?php echo $this->escape($entry->getId()); ?></td>
    </tr>
    <tr>
        <th>first name</th>
        <td><?php echo $this->escape($entry->getFirstName()); ?></td>
    </tr>
    <tr>
        <th>last name</th>
        <td><?php echo $this->escape($entry->getLastName()); ?></td>
    </tr>
    <tr>
        <th>email</th>
        <td><?php echo $this->escape($entry->getEmail()); ?></td>
    </tr>
    <tr>
        <th>age</th>
        <td><?php echo $this->escape($entry->getAge()); ?></td>
    </tr>
    <tr>
        <th>active</th>
        <td><?php echo $this->escape($entry->getActive()); ?></td>
    </tr>
</table>
<a href="/customer/list">back to list</a>
```

**views/scripts/customer/create.phtml & views/scripts/customer/edit.phtml**

```php
<?php /* @var $this Zend_View */ ?>
<?php /* @var $form Application_Form_CustomerForm */ $form = $this->form;?>

<a href="/customer/list">back to list</a>
<?php echo $form->render()?>
<a href="/customer/list">back to list</a>
```
