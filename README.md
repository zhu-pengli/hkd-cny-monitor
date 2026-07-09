# 港币兑人民币汇率监测系统

这个仓库用于监测 `HKD -> CNY`，目标是帮助你在港币换人民币时接近期内高点。系统关注的是 `1 HKD = 多少 CNY`，数值越高，港币换人民币越划算。

## 功能

- 拉取 HKD/CNY 日度参考汇率。
- 计算近期高点、低点、当前分位、7 日/30 日变化。
- 判断是否进入“近期高位区间”。
- 按你的港币金额估算可换人民币，以及距离近期高点少换多少。
- GitHub Actions 工作日北京时间 18:15 自动运行。
- 触发高位时自动开/更新 GitHub Issue，并发送邮件提醒。

## GitHub 上直接运行

打开仓库：

```text
https://github.com/zhu-pengli/hkd-cny-monitor
```

手动运行：

```text
Actions -> HKD/CNY Monitor -> Run workflow
```

自动运行：

```text
工作日北京时间 18:15
```

## 邮件提醒设置

GitHub Actions 发邮件需要一个发信邮箱的 SMTP 授权码。推荐直接用你的 126 邮箱发给自己。

在 GitHub 仓库中进入：

```text
Settings -> Secrets and variables -> Actions -> New repository secret
```

添加这三个必填 Secrets：

```text
ALERT_EMAIL_TO=你的收件邮箱
SMTP_USER=你的发信邮箱
SMTP_PASSWORD=你的邮箱 SMTP 授权码
```

可选 Secrets，默认已经按 126 邮箱配置好，一般不用填：

```text
SMTP_HOST=smtp.126.com
SMTP_PORT=465
```

注意：`SMTP_PASSWORD` 通常不是邮箱登录密码，而是邮箱设置里开启 SMTP/POP3 后生成的“客户端授权码”。

## 怎么判断是否该换

核心看 `data/summary.md` 或 GitHub Actions 的运行摘要：

- `信号：高位提醒`：进入配置的近期高位区间，可以重点比较银行/券商实时买入价，考虑分批换汇。
- `信号：接近高位`：已经偏高但未完全触发提醒，可以继续观察。
- `信号：继续等待`：距离近期高点还有空间。

## 配置

编辑 `config.json`：

```json
{
  "amount_hkd": 100000,
  "lookback_days": 90,
  "near_high_threshold_pct": 0.3,
  "alert_min_percentile": 90,
  "target_rate": null,
  "bank_spread_pct": 0.15
}
```

关键字段：

- `amount_hkd`：准备换出的港币金额。
- `lookback_days`：判断近期高点的窗口，默认 90 个报价日。
- `near_high_threshold_pct`：距离窗口高点多少以内算接近高点，默认 `0.3%`。
- `alert_min_percentile`：当前报价至少处于近期分位的多少以上才提醒，默认 `90%`。
- `target_rate`：你自己的心理价位，例如 `0.93`；填 `null` 则只用近期高点规则。
- `bank_spread_pct`：估算银行/券商价差，默认 `0.15%`，仅用于粗略净额估算。

## 高位提醒规则

默认触发条件：

```text
最新 HKD/CNY 距离 90 日高点 <= 0.3%
并且
当前分位 >= 90%
```

如果设置了 `target_rate`，达到心理价位也会触发提醒。

## 本地运行

```powershell
python monitor.py --print-summary
python -m unittest discover
```

## 数据源和注意事项

数据源是 [Frankfurter exchange-rate API](https://frankfurter.dev/)，适合做日度参考汇率监测。实际换汇前请以银行、券商或持牌机构的实时买入价、手续费和额度规则为准。

这个系统是监测和提醒工具，不构成投资、税务或法律建议。
