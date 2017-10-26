---
layout: post
title: WordPress custom post content widget

tags: [php, plugin, widget, wordpress, register_post_type, get_post, register_sidebar, dynamic_sidebar, wp_query]
---

Задача
------

В WP необходимо создать отдельный тип записей и виджет для вывода их контента в сайтбаре. Виджет при добавлении в сайт будет предлагать в виде дропдауна список кастомных записей для выбора.

Шаг 1. Создание кастомного типа
-------------------------------

В `functions.php` необходимо прописать следующее:

```php
function register_post_type_txt() {
    register_post_type('txt',array(
            'label' => __('Text'),
            'public' => true,
            'supports' => array('title', 'editor')
    ));
}
add_action('init', 'register_post_type_txt');
```

После чего в админке появится новосозданный кастомный тип записей

![screenshot](/images/wp/image01.png)

Подробнее о создании кастомных полей: [register_post_type](http://codex.wordpress.org/Function_Reference/register_post_type)

Шаг 2. Вывод содержимого поста в теме
-------------------------------------

Где угодно в файлах темы (я для примера прописал в `header.php`) прописать следующее:

```php
<?php
$postId = 50;
$queried_post = get_post($postId);
$title = $queried_post->post_title;
$content = $queried_post->post_content;
echo _e($title);
echo '<br />';
echo _e($content);
?>
```

Примечание: в функцию `get_post` - обязательно должна передаваться переменная, иначе получим эксепшн.

Подробнее о ф-ии [get_post](http://codex.wordpress.org/Function_Reference/get_post)

Примечание: в проекте для которого все это делалось, юзается плагин _qTranslate_ для создания многоязычного сайта, этот плагин записывает через комменты все переводы в один пост, для того чтобы отобразить нужный, необходимо прогнать текст через ф-ию `_e()`

Эта заготовка будет использоваться плагином, для вывода содержимого поста.

Шаг 3. Создание кастомных сайтбаров
-----------------------------------

В файле `functions.php` необходимо прописать следующее:

```php
if ( function_exists('register_sidebar') ) {

    register_sidebar(array(
        'name'  => 'home_page_text',
        'before_widget' => '',
        'after_widget' => '',
        'before_title' => '',
        'after_title' => '',
    ));

}
```

Тем самым мы добавляем новый сайтбар для размещения виджетов, соотв. в дальнейшем плодим этих сайдтбаров, столько, сколько нам надо.

После добавления этого кода на странице виджетов должен появиться новый сайтбар:

![screenshot](/images/wp/image11.png)

Шаг 4. Вывод кастомных сайтбаров
--------------------------------

Где угодно в файлах темы (я для примера пробывал в `header.php`) прописываем следующее:

```php
<?php dynamic_sidebar( 'home_page_text' )?>
```

Эта ф-ия выведет содержимое кастомного сайтбара, подробнее: [dynamic_sidebar](http://codex.wordpress.org/Function_Reference/dynamic_sidebar)

Шаг 5. Создание виджета (собираем все в кучу)
---------------------------------------------

Описание [create wordpress widgets](http://www.lonewolfdesigns.co.uk/create-wordpress-widgets/)

Код `/wp-content/plugins/custom-post-type-widget.php`:

```php
<?php
/**
    * Plugin Name: Custom Post Type Widget
    * Plugin URI: http://photowizard.com
    * Description: Widget to display custom post content
    * Version: 0.1
    * Author: mac
    * Author URI: http://photowizard.com.ua
    */

add_action( 'widgets_init', 'widgets_init_custom_post_type_widget' );

function widgets_init_custom_post_type_widget() {
    register_widget( 'Custom_Post_Type_Widget' );
}

class Custom_Post_Type_Widget extends WP_Widget {

    function Custom_Post_Type_Widget() {
        parent::WP_Widget(false, $name = 'Custom_Post_Type_Widget');
    }

    function widget( $args, $instance ) {
        extract( $args );

        $txtId = isset($instance['txtId']) ? $instance['txtId'] : false;

        echo $before_widget;
        if ( $txtId ) {
            $queried_post = get_post($txtId);
            $content = $queried_post->post_content;
            echo _e($content);
        }
        echo $after_widget;
    }

    function update( $new_instance, $old_instance ) {
        $instance = $old_instance;
        $instance['txtId'] = $new_instance['txtId'];
        return $instance;
    }

    function form( $instance ) {
        $txtId = isset($instance['txtId']) ? $instance['txtId'] : 0;
        $opts = '';
        wp_reset_query();
        $my_query = null;
        $my_query = new WP_Query(array(
            'post_type' => 'txt',
            'post_status' => 'publish',
            'posts_per_page' => -1,
            'caller_get_posts'=> 1
        ));
        if( $my_query->have_posts() ) {
            while ($my_query->have_posts()) : $my_query->the_post();
                $sel = ($txtId == get_the_ID()) ? ' selected="selected" ' : '';
                $opts = $opts . '<option ' . $sel . ' value="' . get_the_ID() . '">' . get_the_title() . '</option>';
            endwhile;
        }
        wp_reset_query();

        echo '<p><select id="' . $this->get_field_id( 'txtId' ) . '" name="' . $this->get_field_name( 'txtId' ) . '" class="widefat" style="width:100%;"><option value="0">- select -</option>' . $opts . '</select></p>';
    }
}
?>
```


Примечание: Очень важно в форме правильно использовать методы `$this->get_field_id` и `$this->get_field_name` иначе данные не будут сохранятся и будет куча гемороя.

![screenshot](/images/wp/image21.png)
