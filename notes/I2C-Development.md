# Hikey I2C Development on Zircon



Files: 

- `zircon/system/dev/board/hikey960/hikey960-i2c.c`
- `zircon/system/dev/board/hikey960/hikey960-i2c-test.c`     <-- DAS da als Vorlage?
- `zircon/system/ulib/ddk/include/ddk/protocol/i2c-lib.h`
- `zircon/system/dev/board/hikey960/hikey960-i2c.c`               <-- Board specific I2C stuff, channels, ...
-  18 #include <ddk/protocol/i2c.h>
   19 #include <ddk/protocol/i2c-lib.h>
   20 #include <ddk/protocol/platform/device.h>

PDEV_DID_HIKEY960_I2C_TEST = 1 



Example Impls??    `ag i2c_write_read`

dev/display/hikey-display/adv7533.c                                                <-- Hikey Display I2C

dev/input/i2c-hid/i2c-hid.c

dev/light/ams-light/tcs3400.cpp

system/dev/rtc/nxp/pcf8563-rtc.c

system/dev/display/led2472g/led2472g.c

system/dev/board/gauss/gauss-led.c 		<---- use this one!

 zx_status_t status = i2c_write_read(&gauss_led->channel, write_buf, write_len,
171                                       read_len, i2c_complete, dev);

 if (device_get_protocol(parent, ZX_PROTOCOL_I2C,
504                             &gauss_led->i2c) != ZX_OK) {
505         free(gauss_led);
506         return ZX_ERR_NOT_SUPPORTED;
507     }
508 
509     zx_status_t status = i2c_get_channel(&gauss_led->i2c, 0,
510                                          &gauss_led->channel);



ulib/ddktl/include/ddktl/i2c-channel.h ??



board/astro/astro-i2c.c vs astro-light.c     -> tcs3400.c ? pbus_i2c_channel_t tcs3400_light_i2c[]

astro-backlight.c
15:static const pbus_i2c_channel_t backlight_i2c_channels[] = {
17:        .bus_id = ASTRO_I2C_3,
18:        .address = I2C_BACKLIGHT_ADDR,              <--- 0x2c
27:    .i2c_channel_list = backlight_i2c_channels,
28:    .i2c_channel_count = countof(backlight_i2c_channels),



tcs3400.h:

60     i2c_protocol_t i2c_ TA_GUARDED(i2c_lock_);





 20     // Performs typical i2c Read: writes device register address (1 byte) followed
 21     // by len reads into buf.



if (client != NULL) {
​		i2cRet = i2c_smbus_write_byte_data (rgb1313M1_client, 0, 0);
​	        if (i2cRet < 0) {printk (KERN_INFO "rgb1313M1 failed init rgb p1, ret val =%d \n", i2cRet); }
​       		i2cRet = i2c_smbus_write_byte_data (rgb1313M1_client, 1, 0);
​        	if (i2cRet < 0) {printk (KERN_INFO "rgb1313M1 failed init rgb p2, ret val =%d \n", i2cRet); }
​        	i2cRet = i2c_smbus_write_byte_data (rgb1313M1_client, 0x08, 0xAA);
​	       	if (i2cRet < 0) {printk (KERN_INFO "rgb1313M1 failed init rgb p3, ret val =%d \n", i2cRet); }
​	}



Linux: i2c_smbus_write_byte_data vs Zircon



void Alc5514Device::WriteReg(uint32_t addr, uint32_t val) {
 34     uint32_t buf[2];
 35     buf[0] = htobe32(addr);
 36     buf[1] = htobe32(val);
 37     zx_status_t status = i2c_write_sync(&i2c_, buf, sizeof(buf));
 38     if (status != ZX_OK) {
 39         zxlogf(ERROR, "alc5514: could not write reg addr/val: 0x%08x/0x%08x status: %d\n", addr,
 40                val, status);
 41     }
 42 
 43     zxlogf(SPEW, "alc5514: register 0x%08x write 0x%08x\n", addr, val);
 44 }





[00007.256] 01984.02473> grove: reg: 16, if: 0
[00007.256] 01984.02473> grove: write went fine, dump:
[00007.256] 01984.02473> ########################
[00007.256] 01984.02473> i2c_dw_dumpstate
[00007.256] 01984.02473> ########################
[00007.256] 02596.02625> usb_dev_bind
[00007.256] 01678.01763> vc: new display device /dev/class/display-controller/000/virtcon
[00007.257] 01984.02473> DW_I2C_ENABLE_STATUS = 0x1
[00007.258] 01984.02473> DW_I2C_ENABLE = 0x1
[00007.259] 01984.02473> DW_I2C_CON = 0x65
[00007.260] 01984.02473> DW_I2C_TAR = 0x62
[00007.261] 01984.02473> DW_I2C_HS_MADDR = 0x1
[00007.262] 01984.02473> DW_I2C_SS_SCL_HCNT = 0x87
[00007.263] 01984.02473> DW_I2C_SS_SCL_LCNT = 0x9f
[00007.264] 01984.02473> DW_I2C_FS_SCL_HCNT = 0x1a
[00007.265] 01984.02473> DW_I2C_FS_SCL_LCNT = 0x32
[00007.267] 01984.02473> DW_I2C_INTR_MASK = 0x254
[00007.268] 01984.02473> DW_I2C_RAW_INTR_STAT = 0x10
[00007.269] 01984.02473> DW_I2C_RX_TL = 0x3
[00007.270] 01984.02473> DW_I2C_TX_TL = 0x0
[00007.271] 01984.02473> DW_I2C_STATUS = 0x6
[00007.272] 01984.02473> DW_I2C_TXFLR = 0x0
[00007.273] 01984.02473> DW_I2C_RXFLR = 0x0
[00007.274] 01984.02473> DW_I2C_COMP_PARAM_1 = 0x3f3fee
[00007.275] 01984.02473> DW_I2C_TX_ABRT_SOURCE = 0x0
[00007.293] 01984.02473> grove: reg: 1872, if: 64
[00007.293] 01984.02473> ########################
[00007.293] 01984.02473> i2c_dw_dumpstate
[00007.293] 01984.02473> ########################
[00007.294] 01984.02473> DW_I2C_ENABLE_STATUS = 0x1
[00007.295] 01984.02473> DW_I2C_ENABLE = 0x1
[00007.296] 01984.02473> DW_I2C_CON = 0x65
[00007.297] 01984.02473> DW_I2C_TAR = 0x62
[00007.299] 01984.02473> DW_I2C_HS_MADDR = 0x1
[00007.300] 01984.02473> DW_I2C_SS_SCL_HCNT = 0x87
[00007.301] 01984.02473> DW_I2C_SS_SCL_LCNT = 0x9f
[00007.302] 01984.02473> DW_I2C_FS_SCL_HCNT = 0x1a
[00007.303] 01984.02473> DW_I2C_FS_SCL_LCNT = 0x32
[00007.304] 01984.02473> DW_I2C_INTR_MASK = 0x254
[00007.305] 01984.02473> DW_I2C_RAW_INTR_STAT = 0x750
[00007.306] 01984.02473> DW_I2C_RX_TL = 0x3
[00007.307] 01984.02473> DW_I2C_TX_TL = 0x0
[00007.308] 01984.02473> DW_I2C_STATUS = 0x6
[00007.310] 01984.02473> DW_I2C_TXFLR = 0x0
[00007.311] 01984.02473> DW_I2C_RXFLR = 0x0
[00007.312] 01984.02473> DW_I2C_COMP_PARAM_1 = 0x3f3fee
[00007.313] 01984.02473> DW_I2C_TX_ABRT_SOURCE = 0x1
[00007.313] 01984.02473> grove: we runned into irq



## Needed

- board file like board/gauss/gauss.c or board/astro/astro-light.c  defining the .did  -> last instance for matching
- header definition for the value used in .did (system/ulib/ddk/include/ddk/platform-defs.h?)
- matching (using aboves above -> should bind only to the wished device!)
- bind to i2c protocol



- i2c_write_sync returns -21 -> 

  // ZX_ERR_TIMED_OUT: The time limit for the operation elapsed before
   69 // the operation completed.
   70 #define ZX_ERR_TIMED_OUT (-21)

  -> TODO remove write operation from bind?