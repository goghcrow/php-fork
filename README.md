# php多进程任务

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
