---
layout: post
title: Zend simplest auth

tags: [adapter, auth, authenticate, autoincrement, database, db, dbadapter, id, identity, login, noautoincrement, noid, php, sequence, zend]
---

**Data base:**

```sql
CREATE TABLE IF NOT EXISTS credential (
    email varchar(100) NOT NULL,
    password varchar(100) NOT NULL,
    PRIMARY KEY (email)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

INSERT INTO credential VALUES('alexandrm@rabota.ua', '123');
```

**Application.ini:**

```ini
resources.db.adapter = "PDO_MYSQL"
resources.db.params.host = "localhost"
resources.db.params.username = "root"
resources.db.params.password = ""
resources.db.params.dbname = "simpleauth_zf"
resources.db.isDefaultTableAdapter = true
```

**User Controller:**

```php
<?php
class UserController extends Zend_Controller_Action
{
    public function preDispatch()
    {
        if (Zend_Auth::getInstance()->hasIdentity()) {
            // If the user is logged in, we don't want to show the login form;
            // however, the logout action should still be available
            if ('logout' != $this->getRequest()->getActionName()) {
                $this->_helper->redirector('index', 'index');
            }
        }
    }

    public function indexAction()
    {
        // action body
    }

    public function loginAction()
    {
        $form = new Application_Form_Login(array(
            'action' => '/user/login',
            'method' => 'post',
        ));

        if ($this->getRequest()->isPost()) {
            if ($form->isValid($_POST)) {

                $authAdapter = new Zend_Auth_Adapter_DbTable(Zend_Db_Table::getDefaultAdapter(), 'credential', 'email', 'password');

                $authAdapter->setIdentity($form->getValue('email'));
                $authAdapter->setCredential($form->getValue('password'));

                $result = Zend_Auth::getInstance()->authenticate($authAdapter);

                if (!$result->isValid()) {
                    var_dump('FAILURE');
                } else {
                    $this->_helper->redirector('index','index');
                }

                /*switch ($result->getCode()) {
                    case Zend_Auth_Result::FAILURE_IDENTITY_NOT_FOUND:
                        var_dump('FAILURE_IDENTITY_NOT_FOUND');
                        break;
                    case Zend_Auth_Result::FAILURE_CREDENTIAL_INVALID:
                        var_dump('FAILURE_CREDENTIAL_INVALID');
                        break;
                    case Zend_Auth_Result::SUCCESS:
                        $this->_helper->redirector('index','index');
                        break;
                    default:
                        break;
                }*/
            }
        }

        $this->view->form = $form;
    }

    public function logoutAction()
    {
        Zend_Auth::getInstance()->clearIdentity();
        $this->_helper->redirector('index','index');
    }

}
```

**Index Controoler:**

```php
<?php

class IndexController extends Zend_Controller_Action
{
    public function indexAction()
    {
        if (Zend_Auth::getInstance()->hasIdentity()) {
            $username = Zend_Auth::getInstance()->getIdentity();
            $profile = 'Welcome, ' . $username . ' <a href="/user/logout">logout</a>';
        } else {
            $profile = '<a href="/user/login">Login</a>';
        }

        $this->view->profile = $profile;
    }
}
```

**Login form:**

```php
<?php

//zf create form Login
class Application_Form_Login extends Zend_Form
{

    public function init()
    {
        $email = new Zend_Form_Element_Text('email');
        $email->setLabel('Email:');
        $email->setRequired();
        $email->addValidator(new Zend_Validate_EmailAddress());
        $email->addFilters(array(
            new Zend_Filter_StringTrim(),
            new Zend_Filter_StringToLower(),
        ));
        $email->setAttrib('size', 30);
        $this->addElement($email);

        $password = new Zend_Form_Element_Password('password');
        $password->setLabel('Password:');
        $password->setRequired();
        $password->addFilters(array(
            new Zend_Filter_StringTrim()
        ));
        $password->setAttrib('size', 30);
        $this->addElement($password);

        $this->addElement('submit', 'submit', array('label' => 'Submit'));
    }

}
```

User/login view:

```php
<h3>login</h3>
<?php echo $this->form->render()?>
```

Index/index view:

```php
<h3>Hello World</h3>
<?php echo $this->profile?>
```

Examples copy pasted from:

http://weierophinney.net/matthew/archives/165-Login-and-Authentication-with-Zend-Framework.html

http://framework.zend.com/manual/en/zend.auth.adapter.dbtable.html

If you want create data table adapter, do not forget to add

```php
protected $_sequence = false;
```

to it.

Passwords should be encrypted with some functions like MD5. To do it change code like this:

```php
$authAdapter = new Zend_Auth_Adapter_DbTable(Zend_Db_Table::getDefaultAdapter(), 'credential', 'email', 'password', 'MD5(?)');
```
