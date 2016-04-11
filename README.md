# php多进程任务

~~~ php
namespace xiaofeng\cli;
require_once __DIR__ . DIRECTORY_SEPARATOR . "Fork.php";

error_reporting(E_ALL);
ini_set("error_log", "error.log");

log("start...");
$fork = new Fork();

// 四个进程:
// 第一个儿子睡一秒.第二个儿子睡三秒...返回自己睡眠时间
// 第三个儿子睡了两秒, 等其他两兄弟全部醒来后, 把三个人的睡眠时间求和
// 他爸等三哥俩都睡醒了,从第三个儿子取到总睡眠时间
// 总消耗3秒,求得睡眠时间6秒
$f_one = $fork->task(function() {
	$x = 1;
	sleep($x);
	return $x;
});

$f_another = $fork->task(function() {
	$x = 3;
	sleep($x);
	return $x;
});

$f_add = $fork->task(function() use($f_one, $f_another){
	$x = 2;
	sleep($x);
	return $f_one->combine($f_another, function($r1, $r2) use($x) {
		return $r1 + $r2 + $x;
	});
});


// do something else ...
// 阻塞等结果
log($f_add->get());
~~~

~~~ php
namespace xiaofeng\cli;
require_once __DIR__ . DIRECTORY_SEPARATOR . "Fork.php";

error_reporting(E_ALL);
ini_set("error_log", "error.log");

$w = new Fork();
log("start: " . microtime(true));


// 添加10个并行任务~
$i = 0;
$futures = [];
while($i < 10){
    $futures[] = $w->task(function() {
        sleep(5);
        return microtime(true);
    });
    $i++;
}

/* @var Future $f */
$f = $futures[1];
// 取消任务
$f->cancel();

/* @var Future $f1 */
$f1 = $futures[3];
// 取消任务
$f1->worker()->suicide();

$fret = $w->task(function() {
    sleep(2);
    return str_repeat("=", 10000) . "\n";
});

// 阻塞等待单个任务完成
echo $fret->get();

// 阻塞等待所有任务完成
$w->wait();

// 处理任务返回结果
/* @var $f Future */
foreach($futures as $f) {
    $pid = $f->pid();
    log("[$pid]finished: " . $f->get());
}

// 打印worker状态
$w->worker_status();

~~~
