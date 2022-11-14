---
title: 限制ip接口案例
date: 2021-09-01 09:51:50
tags: 细节
---

## 为什么要开细节类

因为有很多东西，很细微，但又很实用。就会放在细节类。该类的目的是做个记录，既然简单。

## 限制ip接口案例

> 场景

需要针对相同ip用户每分访问接口次数的做限制，防止用户恶意刷。

> 代码

```
/**
 * 接口 ip 限制
 * @param $miniutes 分钟数
 * @param $times 次数
 * @param $api 接口唯一标识
 * @return bool
 */
public function apiIpLimit($miniutes, $times, $api)
{
    $ip = Request::getClientIP(); //获取用户ip

    if ($ip) {
        $key = $api . "_" . $ip;

        $num = Yii::$app->cookie->get($key);
        $num = $num ? $num + 1 : 1; //请求次数加1
        if ($num > $times) {
            Yii::getLogger()->log($key . " exceed limit", Logger::LEVEL_ERROR);
            return false; //超过限制次数直接返回，不重置过期时间
        }

        Yii::$app->cookie->set($key, $num, $miniutes * 60); //未超过限制次数，更新访问次数，并重置过期时间为一分钟
    }

    return true;
}
```

## 解析

> 情景

假设每分钟限制8次请求

> 代码逻辑是否符合需求

上面的代码案例并不是完全依照`每分钟限制8次请求`逻辑处理，因为有重置过期时间。

假设第6，7，8次请求都间隔了40s。按逻辑，第6次和第8次中间间隔了80s，超过一分钟了，但因为访问接口，未超过限制次数时会重置`过期时间`为60s的因素，第6次访问和第8次访问被累加在同一个次数累计中。

> 代码合理性

从合理性上，我觉得是合理的。但逻辑解释应该是**相同ip距离上一次访问时间不超过一分钟，计入次数累计，当次数累计超过限制次数将会返回错误，时间1min**

当然，如果要按目前逻辑实现也很简单，可以存储值`num_day`(累计次数+当天日期`(最小单位:天)`)，然后重置的时间设置为`当前时间戳-第一次的时间戳`。

> 优化点

首先ip这块用户可以通过vpn等手段切换不同ip。

其次，限制次数是设置在cookie用户是可以删除的。

建议：限制用户每天可以请求多少次，记录在redis之类的服务端缓存。

## 限制用户每天可以请求多少次

> 检验

```
// 邀请邮件 校验: 每天(utc时区)限制50封
if ($this->getApiRequestLimitByDay(50, 'bizInviteEmail') === false) {
    Response::echoResult(ErrCode::$ACCOUNT_LIMIT_ERR);
}
```

> 累计次数

```
$this->setApiRequestLimitByDay('bizInviteEmail', $accountId); //发送邮件后更新用户当天发邮件次数
```

> 方法

```
/**
 * 获取接口是否超出每日访问次数限制
 * @param $times 次数
 * @param $api 限制内容唯一标识
 * @param string $accountId 用户主账号id
 * @return bool
 */
public function getApiRequestLimitByDay($times, $api, $accountId = "")
{
    if(empty($accountId)) {
        $accountId = \Yii::$app->bizUser->accountId;
    }

    if ($accountId) {
        $day = date('Y_m_d',time());
        $key = $api . "_" . $accountId . "_" . $day;

        $redis = Factory::getUserRedisClient();
        $num = $redis->get($key);
        $num = $num ? $num + 1 : 1;

        if ($num > $times) {
            Yii::getLogger()->log($key . " exceed limit", Logger::LEVEL_ERROR);
            return false;
        }
    }

    return true;
}

/**
 * 更新(累加)接口每日访问次数
 * @param $api 限制内容唯一标识
 * @param string $accountId 用户主账号id
 */
public function setApiRequestLimitByDay($api, $accountId = "")
{
    if(empty($accountId)) {
        $accountId = \Yii::$app->bizUser->accountId;
    }

    if ($accountId) {
        $day = date('Y_m_d',time());
        $key = $api . "_" . $accountId . "_" . $day;

        $redis = Factory::getUserRedisClient();
        $num = $redis->get($key);
        $num = $num ? $num + 1 : 1;

        try {
            $redis->setex($key, 86400, $num); //保存一天后过期
        } catch (\Exception $e) {
            LogService::logError("set $ownerId $key exception : " . $e->getMessage());
        }
    }
}
```