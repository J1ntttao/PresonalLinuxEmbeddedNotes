# 1. 申请 PE10 (编号 138)
echo 138 > /sys/class/gpio/export

# 2. 设置为输出
echo out > /sys/class/gpio/gpio138/direction

# 3. 拉高电平 (High)
echo 1 > /sys/class/gpio/gpio138/value

# 4. 查看当前状态
cat /sys/class/gpio/gpio138/value

echo GDADC0 > /sys/class/gpio/export

echo 193 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio193/direction
echo 1 > /sys/class/gpio/gpio193/value
cat /sys/class/gpio/gpio193/value


///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////



1.（向系统申请使用 pwm6）。
echo 6 > /sys/class/pwm/pwmchip0/export
2. 定下节奏（设置周期）：
echo 1000000 > /sys/class/pwm/pwmchip0/pwm6/period
3. 开启一半亮度（设置占空比）：
echo 500000 > /sys/class/pwm/pwmchip0/pwm6/duty_cycle
4. 确认极性：
echo "normal" > /sys/class/pwm/pwmchip0/pwm6/polarity
5. 通电起飞：
echo 1 > /sys/class/pwm/pwmchip0/pwm6/enable
【见证奇迹的时刻】 现在灯亮着，我们不关灯，直接动态修改占空比：
● 敲下 echo 100000 > .../duty_cycle，灯瞬间变暗（10%亮度）。
● 敲下 echo 900000 > .../duty_cycle，灯瞬间耀眼（90%亮度）。


pwm呼吸灯：
cat > /mnt/UDISK/pwmlight.sh << 'EOF'
#!/bin/sh
# 呼吸灯脚本 - 兼容busybox ash

# 配置参数
PWM_CHIP="pwmchip0"
PWM_NUM="6"
PERIOD="1000000"
MIN_DUTY="0"
MAX_DUTY="1000000"
STEP="20000"
DELAY="0.01"

# 清理函数
cleanup() {
    echo "正在清理..."
    echo 0 > /sys/class/pwm/$PWM_CHIP/pwm$PWM_NUM/enable 2>/dev/null
    echo $PWM_NUM > /sys/class/pwm/$PWM_CHIP/unexport 2>/dev/null
    exit 0
}

# 设置信号处理
trap cleanup INT TERM

# 导出PWM（检查是否已存在）
if [ ! -d "/sys/class/pwm/$PWM_CHIP/pwm$PWM_NUM" ]; then
    echo $PWM_NUM > /sys/class/pwm/$PWM_CHIP/export 2>/dev/null
    sleep 1
fi

# 等待PWM设备就绪
sleep 1

# 设置PWM参数
echo $PERIOD > /sys/class/pwm/$PWM_CHIP/pwm$PWM_NUM/period
echo "normal" > /sys/class/pwm/$PWM_CHIP/pwm$PWM_NUM/polarity
echo 1 > /sys/class/pwm/$PWM_CHIP/pwm$PWM_NUM/enable

echo "呼吸灯启动中... (按Ctrl+C停止)"

# 呼吸灯效果
duty=$MIN_DUTY
direction=1

while true; do
    echo $duty > /sys/class/pwm/$PWM_CHIP/pwm$PWM_NUM/duty_cycle
    
    /bin/sleep $DELAY
    
    if [ $direction -eq 1 ]; then
        duty=$((duty + STEP))
        if [ $duty -ge $MAX_DUTY ]; then
            duty=$MAX_DUTY
            direction=0
        fi
    else
        duty=$((duty - STEP))
        if [ $duty -le $MIN_DUTY ]; then
            duty=$MIN_DUTY
            direction=1
        fi
    fi 
done
EOF


///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////


GPADC0，按下按钮接地:
root@TinaLinux:/# cat /sys/bus/iio/devices/iio\:device0/in_voltage0_raw
558
root@TinaLinux:/# cat /sys/bus/iio/devices/iio\:device0/in_voltage0_raw
0

