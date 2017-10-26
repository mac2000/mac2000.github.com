---
layout: post
title: php нейронная сеть

tags: [ai, neural, neuralnetwork, php]
---

Важно, на вход могут подаваться только 1,0

Не забываем менять размерность.

```php
<?
/**
    * Simple Perceptron.
    *
    */
class Perceptron
{
    private $W;     // weights
    private $size;
    private $porog;

    /**
        * Asking perceptron.
        * Activation function
        *
        * @param array $vector
        * @return int
        */
    public function ask($vector) {
        $sum = 0;
        for($i=0;$i<count($vector);$i++) {
            $sum += $vector[$i]*$this->W[$i];
        }
        if($sum > $this->porog) return 1;
        return -1;
    }

    /**
        * Constructor
        * arg - dimensions of perceptron
        *
        * @param int $n
        */
    public function __construct($n) {
        $this->size    = $n;
        $this->porog    = 100;
        $this->init_weight();
    }

    /**
        * Initializing start weights.
        * Random
        *
        */
    public function init_weight() {
        for($i=0;$i<$this->size;$i++) $this->W[] = rand(0, 10);
    }

    /**
        * Saving to file
        * If file exists - overwrite
        *
        * @param string $filename
        */
    public function weight_save($filename) {
        $serialize = serialize($this->W);
        fwrite( fopen($filename,"w"), $serialize);
    }

    /**
        * Load weights from file
        *
        *
        * @param string $filename
        */
    public function weight_load($filename) {
        $this->W = unserialize(file_get_contents($filename));
    }

    public function teach($vector, $d) {
        if($d!=$this->ask($vector)) {
            // teach
            for($i=0;$i<$this->size;$i++) {
                $this->W[$i] += $d*$vector[$i];
            }
        }
    }
}

/**
    * Example
    */
$filename = 'w1.txt';

/**
    * Perceptron will guest squares and lines.
    * Notice that we are asking about figures,
    * that was not used in teaching process.
    */

$neural = new Perceptron(64); // matrix will be 8x8, dimension 64.

if(!isset($filename)) {
    /**
        * Learning squares
        */
    $v1 = array(1, 1, 1, 1, 1, 1, 1, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 1, 1, 1, 1, 1, 1, 1);
    $neural->teach( $v1, 1 );

    $v1 = array(0, 1, 1, 1, 1, 1, 1, 1,
                0, 1, 0, 0, 0, 0, 0, 1,
                0, 1, 0, 0, 0, 0, 0, 1,
                0, 1, 0, 0, 0, 0, 0, 1,
                0, 1, 0, 0, 0, 0, 0, 1,
                0, 1, 0, 0, 0, 0, 0, 1,
                0, 1, 0, 0, 0, 0, 0, 1,
                0, 1, 1, 1, 1, 1, 1, 1);
    $neural->teach( $v1, 1 );

    $v1 = array(0, 0, 0, 0, 0, 0, 0, 0,
                1, 1, 1, 1, 1, 1, 1, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 1, 1, 1, 1, 1, 1, 1);
    $neural->teach( $v1, 1 );

    $v1 = array(0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                1, 1, 1, 1, 1, 1, 1, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 1, 1, 1, 1, 1, 1, 1);
    $neural->teach( $v1, 1 );

    $v1 = array(0, 0, 1, 1, 1, 1, 1, 1,
                0, 0, 1, 0, 0, 0, 0, 1,
                0, 0, 1, 0, 0, 0, 0, 1,
                0, 0, 1, 0, 0, 0, 0, 1,
                0, 0, 1, 0, 0, 0, 0, 1,
                0, 0, 1, 0, 0, 0, 0, 1,
                0, 0, 1, 1, 1, 1, 1, 1,
                0, 0, 0, 0, 0, 0, 0, 0);
    $neural->teach( $v1, 1 );

    $v1 = array(1, 1, 1, 1, 1, 1, 1, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 0, 0, 0, 0, 0, 0, 1,
                1, 1, 1, 1, 1, 1, 1, 1);
    $neural->teach( $v1, 1 );

    /**
        * Teaching lines.
        */
    $v1 = array(1, 1, 1, 1, 1, 1, 1, 1,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0);
    $neural->teach( $v1,-1 );

    $v1 = array(0, 0, 0, 0, 0, 0, 0, 1,
                0, 0, 0, 0, 0, 0, 1, 0,
                0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 1, 0, 0, 0,
                0, 0, 0, 1, 0, 0, 0, 0,
                0, 0, 1, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0,
                1, 0, 0, 0, 0, 0, 0, 0);
    $neural->teach( $v1,-1 );

    $v1 = array(0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                1, 1, 1, 1, 1, 1, 1, 1,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0);
    $neural->teach( $v1, -1 );

    $v1 = array(0, 1, 0, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0);
    $neural->teach( $v1,-1 );

    $v1 = array(1, 0, 0, 0, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0,
                0, 0, 1, 0, 0, 0, 0, 0,
                0, 0, 0, 1, 0, 0, 0, 0,
                0, 0, 0, 0, 1, 0, 0, 0,
                0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 0, 1, 0,
                0, 0, 0, 0, 0, 0, 0, 1);
    $neural->teach( $v1,-1 );

    $v1 = array(0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 1, 0, 0);
    $neural->teach( $v1,-1 );

    /**
        * End teaching.
        * Write weights to file.
        */
    $neural->weight_save("w1.txt");
} else {
    // If we already learned something - load weight from file.
    $neural->weight_load($filename);;
}

// Square, that was not been in teaching program
$v1 = array(0, 0, 1, 1, 1, 1, 1, 1,
            0, 0, 1, 0, 0, 0, 0, 1,
            0, 0, 1, 0, 0, 0, 0, 1,
            0, 0, 1, 0, 0, 0, 0, 1,
            0, 0, 1, 0, 0, 0, 0, 1,
            0, 0, 1, 0, 0, 0, 0, 1,
            0, 0, 1, 0, 0, 0, 0, 1,
            0, 0, 1, 1, 1, 1, 1, 1);
echo $neural->ask( $v1 )==1?"square":"line";
echo "<br />";
// Another square
$v1 = array(0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 1, 1, 1, 1, 1, 0,
            0, 0, 1, 0, 0, 0, 1, 0,
            0, 0, 1, 0, 0, 0, 1, 0,
            0, 0, 1, 0, 0, 0, 1, 0,
            0, 0, 1, 0, 0, 0, 1, 0,
            0, 0, 1, 1, 1, 1, 1, 0,
            0, 0, 0, 0, 0, 0, 0, 0);
echo $neural->ask( $v1 )==1?"square":"line";
echo "<br />";

// Now what about lines
$v1 = array(0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0, 0,
            1, 1, 1, 1, 1, 1, 1, 1,
            0, 0, 0, 0, 0, 0, 0, 0);
echo $neural->ask( $v1 )==1?"square":"line";
echo "<br />";

// Asking about 5 cells length line rather that 8.
$v1 = array(0, 0, 0, 0, 1, 0, 0, 0,
            0, 0, 0, 1, 0, 0, 0, 0,
            0, 0, 1, 0, 0, 0, 0, 0,
            0, 1, 0, 0, 0, 0, 0, 0,
            1, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0, 0,
            0, 0, 0, 0, 0, 0, 0, 0);

echo $neural->ask( $v1 )==1?"square":"line";
echo "<br />";
```