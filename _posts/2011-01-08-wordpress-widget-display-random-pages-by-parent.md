---
layout: post
title: WordPress widget display random pages by parent

tags: [menu, php, random, widget, wordpress, WP_Query]
---

Widget must display links to random pages by parent page id

```php
<?php
/*
Plugin Name: RndByPID
Plugin URI: http://wordpress.org/
Description: Plugin4murka
Author: mac
Version: 1
Author URI: http://ma.tt/
*/

add_action( 'widgets_init', 'widgets_init_RndByPID' );

function widgets_init_RndByPID() {
    register_widget( 'RndByPID_Widget' );
}

class RndByPID_Widget extends WP_Widget {

    function RndByPID_Widget() {
        parent::WP_Widget(false, $name = 'RndByPID_Widget');
    }

    function widget( $args, $instance ) {
        extract( $args );

        $pid = isset($instance['pid']) ? $instance['pid'] : false;
        $nop = isset($instance['nop']) ? $instance['nop'] : 0;
        $tit = isset($instance['tit']) ? $instance['tit'] : 'Random articles';

        echo $before_widget;

        echo '<div class="widget-title"><h3>' . $tit . '</h3></div>';

        if ( $pid ) {
            $res = '<div class="menu-custom-container"><ul id="menu-custom" class="menu">';
            wp_reset_query();
            $my_query = null;
            $my_query = new WP_Query(array(
                'post_type' => 'page',
                'post_status' => 'publish',
                'post_parent' => $pid,
                'posts_per_page' => $nop,
                'orderby' => 'rand',
                'caller_get_posts'=> 1
            ));
            if( $my_query->have_posts() ) {
                while ($my_query->have_posts()) : $my_query->the_post();
                    $res = $res . '<li id="menu-item-' . get_the_ID() . '" class="menu-item menu-item-type-custom menu-item-' . get_the_ID() . '">';
                    $res = $res . '<a href="' . get_permalink() . '">' . get_the_title() . '</a>';
                    $res = $res . '</li>';
                endwhile;
            }
            wp_reset_query();
            $res = $res . '</ul></div>';

            echo $res;
        }
        echo $after_widget;
    }

    function update( $new_instance, $old_instance ) {
        $instance = $old_instance;
        $instance['pid'] = $new_instance['pid'];
        $instance['nop'] = $new_instance['nop'];
        $instance['tit'] = $new_instance['tit'];
        return $instance;
    }

    function form( $instance ) {
        //title
        $tit = isset($instance['tit']) ? $instance['tit'] : 'Random articles';
        echo '<p><label for="' . $this->get_field_id( 'tit' ) . '">Title:</label><input id="' . $this->get_field_id( 'tit' ) . '" name="' . $this->get_field_name( 'tit' ) . '" class="widefat" style="width:100%;" value="' . $tit . '" /></p>';

        //parent page
        $pid = isset($instance['pid']) ? $instance['pid'] : 0;
        $opts = '';
        wp_reset_query();
        $my_query = null;
        $my_query = new WP_Query(array(
            'post_type' => 'page',
            'post_status' => 'publish',
            'posts_per_page' => -1,
            'caller_get_posts'=> 1
        ));
        if( $my_query->have_posts() ) {
            while ($my_query->have_posts()) : $my_query->the_post();
                $sel = ($pid == get_the_ID()) ? ' selected="selected" ' : '';
                $opts = $opts . '<option ' . $sel . ' value="' . get_the_ID() . '">' . get_the_title() . '</option>';
            endwhile;
        }
        wp_reset_query();

        echo '<p><label for="' . $this->get_field_id( 'pid' ) . '">Page parent:</label><select id="' . $this->get_field_id( 'pid' ) . '" name="' . $this->get_field_name( 'pid' ) . '" class="widefat" style="width:100%;"><option value="0">- select -</option>' . $opts . '</select></p>';

        //number of pages to display
        $nop = isset($instance['nop']) ? $instance['nop'] : 0;
        $opts = '';
        for($i = 0; $i < 100; $i++) {
            $sel = ($nop == $i) ? ' selected="selected" ' : '';
            $opts = $opts . '<option ' . $sel . ' value="' . $i . '">' . $i . '</option>';
        }
        echo '<p><label for="' . $this->get_field_id( 'nop' ) . '">Show posts:</label><select id="' . $this->get_field_id( 'nop' ) . '" name="' . $this->get_field_name( 'nop' ) . '" class="widefat" style="width:100%;"><option value="0">- select -</option>' . $opts . '</select></p>';
    }
}
?>
```

Widget has only 3 params: title, parent page id and number of pages to display.

Output copied from `Custom_Menu` widget so probably widget will be displayed normaly on any theme.
