---
layout: post
title: Timer for console batch operations

tags: [batch, cms, console, countdown, php, shell, time, timeleft, timer]
---

Here is screenshot:

![screenshot](/images/wp/14.png)

Here is code for timer:

```php
<?php
class Timer {

    /**
        * @var float
        */
    protected $_currentIteration = 0;
    /**
        * @var float
        */
    protected $_totalIterationsCount = 0;
    /**
        * @var int
        */
    protected $_strLengthOfTotalIterationsCount = 0;
    /**
        * @var float
        */
    protected $_microtimeStart = 0;
    /**
        * @var float
        */
    protected $_microtimePrev = 0;
    /**
        * @var int
        */
    protected $_percent = 0;
    /**
        * @var float
        */
    protected $_secondsFromStart = 0;
    /**
        * @var float
        */
    protected $_secondsFromPrev = 0;
    /**
        * @var float
        */
    protected $_secondsPerIteration = 0;
    /**
        * @var float
        */
    protected $_secondsTotal = 0;
    /**
        * @var float
        */
    protected $_secondsRemain = 0;

    /**
        * @param float $totalIterationsCount
        */
    public function __construct($totalIterationsCount) {
        $this->_totalIterationsCount = $totalIterationsCount;
        $this->_strLengthOfTotalIterationsCount = strlen($totalIterationsCount);
        $this->_microtimeStart = $this->_getTime();
    }

    public function tick() {
        $now = $this->_getTime();
        $this->_currentIteration++;

        $this->_secondsFromStart = $now - $this->_microtimeStart;
        $this->_secondsFromPrev = $now - $this->_microtimePrev;
        $this->_secondsPerIteration = $this->_secondsFromStart / $this->_currentIteration;
        $this->_secondsTotal = round(($this->_totalIterationsCount / $this->_currentIteration) * $this->_secondsFromStart);
        $this->_secondsRemain = round($this->_secondsTotal - $this->_secondsFromStart);
        $this->_percent = round($this->_currentIteration / ($this->_totalIterationsCount / 100));

        $this->_microtimePrev = $now;

        $info = array();
        //iteration info
        $info[] = sprintf('[%' . $this->_strLengthOfTotalIterationsCount . 's/%s]', $this->_currentIteration, $this->_totalIterationsCount);
        //percent info
        $info[] = sprintf('[%3s', $this->_percent) . '%]';
        //time per item
        $info[] = '[' . $this->_pretifySeconds($this->_secondsPerIteration) . '/' . $this->_pretifySeconds($this->_secondsRemain) . ']';
        //time remain
        //$info[] = '[' . $this->_pretifySeconds($this->_secondsRemain) . ']';
        //time total
        //$info[] = '[' . $this->_pretifySeconds($this->_secondsTotal) . ']';

        return implode(' ', $info);
    }

    private function _pretifySeconds($integer) {
        $res = "";
        $seconds = round(floatval($integer));
        $minutes = 0;
        $hours = 0;
        $days = 0;
        $weeks = 0;

        if ($seconds / 60 >= 1) {
            $minutes = floor($seconds / 60);

            if ($minutes / 60 >= 1) { # Hours
                $hours = floor($minutes / 60);

                if ($hours / 24 >= 1) { #days
                    $days = floor($hours / 24);

                    if ($days / 7 >= 1) { #weeks
                        $weeks = floor($days / 7);

                        if ($weeks >= 2)
                            $res = $weeks . 'w';
                        else
                            $res=$weeks . 'w';
                    } #end of weeks

                    $days = $days - (floor($days / 7)) * 7;

                    if ($weeks >= 1 && $days >= 1)
                        $res = "$res, ";

                    if ($days >= 2)
                        $res = "$res $days" . 'd';

                    if ($days == 1)
                        $res = "$res $days" . 'd';
                } #end of days

                $hours = $hours - (floor($hours / 24)) * 24;

                if ($days >= 1 && $hours >= 1)
                    $res = "$res, ";

                if ($hours >= 2)
                    $res = "$res $hours" . "h";

                if ($hours == 1)
                    $res = "$res $hours" . "h";
            } #end of Hours

            $minutes = $minutes - (floor($minutes / 60)) * 60;

            if ($hours >= 1 && $minutes >= 1)
                $res = "$res, ";

            if ($minutes >= 2)
                $res = "$res $minutes" . "m";

            if ($minutes == 1)
                $res = "$res $minutes" . "m";
        } #end of minutes

        $seconds = round(floatval($integer)) - (floor(floatval($integer) / 60)) * 60;

        if ($minutes >= 1 && $seconds >= 1)
            $res = "$res, ";

        if ($seconds >= 2)
            $res = "$res $seconds" . "s";

        if ($seconds == 1)
            $res = "$res $seconds" . "s";

        return $res;
    }

    private function _getTime() {
        $mtime = microtime();
        $mtime = explode(" ", $mtime);
        return $mtime[1] + $mtime[0];
    }

}
```

Here is usage example:

```php
<?php
require_once('Timer.php');
$timer = new Timer(100);
for ($i = 1; $i <= 100; $i++) {
    echo $timer->tick();
    echo PHP_EOL;
    sleep(rand(1,2));
}
```
