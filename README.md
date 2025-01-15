# [🔁 EEPROM 生成器](https://kubabuda.github.io/EEPROM_generator)

这是一个用于 EtherCAT 设备的基本代码生成工具，使用 [SOES 库](https://github.com/OpenEtherCATsociety/SOES)。

[在线访问此工具](https://kubabuda.github.io/EEPROM_generator)

您可以配置：
- ESC（Ethercat 从设备芯片）
- OD（CANopen 对象字典）条目
- PDO 映射（哪些 OD 对象映射在 TX 和 RX 数据报中）

该工具在 C 源代码、ESI 文件和 EEPROM 内容之间生成一致的数据。

它还会在本地存储中备份您当前的项目。您可以将项目保存为 JSON 文件到硬盘上，稍后恢复，并一次性下载所有文件。

## 限制

- 仅支持单个、非动态的 PDO，分别用于 TX 和 RX
- 某些数据类型可能不受支持

如果您需要更多功能，[RT-Labs](https://rt-labs.com/ethercat/) 提供专业的 IDE - EtherCAT SDK 和培训。

# 开发

欢迎提交拉取请求。

源代码故意保持为纯 JavaScript 文件，因此不需要像 webpack 这样的构建系统或甚至是 Web 服务器。唯一的依赖是网页浏览器，这将确保其未来的兼容性。

## [单元测试](https://kubabuda.github.io/EEPROM_generator/tests.html)

[单元测试](https://kubabuda.github.io/EEPROM_generator/tests.html) 使用 [Jasmine](https://jasmine.github.io)。

## OD 结构

OD 以 JSON 对象形式保存。预期的数据格式：

```js
{
    '1000': { otype: OTYPE.VAR, dtype: DTYPE.UNSIGNED32, name: 'Device Type', value: 0x1389 },
    '1008': { otype: OTYPE.VAR, dtype: DTYPE.VISIBLE_STRING, name: 'Device Name', data: '' },
    '1009': { otype: OTYPE.VAR, dtype: DTYPE.VISIBLE_STRING, name: 'Hardware Version', data: '' },
    '100A': { otype: OTYPE.VAR, dtype: DTYPE.VISIBLE_STRING, name: 'Software Version', data: '' },
    '1018': { otype: OTYPE.RECORD, name: 'Identity Object', items: [
        { name: 'Max SubIndex' },
        { name: 'Vendor ID', dtype: DTYPE.UNSIGNED32, value: 600 },
        { name: 'Product Code', dtype: DTYPE.UNSIGNED32 },
        { name: 'Revision Number', dtype: DTYPE.UNSIGNED32 },
        { name: 'Serial Number', dtype: DTYPE.UNSIGNED32, data: '&Obj.serial' },
    ]},
    '1C00': { otype: OTYPE.ARRAY, dtype: DTYPE.UNSIGNED8, name: 'Sync Manager Communication Type', items: [
        { name: 'Max SubIndex' },
        { name: 'Communications Type SM0', value: 1 },
        { name: 'Communications Type SM1', value: 2 },
        { name: 'Communications Type SM2', value: 3 },
        { name: 'Communications Type SM3', value: 4 },
    ]},
}   
```

生成器的 OD 模型有 4 个部分：

- `sdo`，未映射到 PDO
- `txpdo`，映射到 TXPDO（SM3）。预期格式（对于 OTYPE VAR）：
```js
{
    '6000': { otype: OTYPE.VAR, dtype: DTYPE.UNSIGNED32, name: 'TXPDO', value: 0x1389, pdo_mappings: ['tx'] },
}
```
- `rxpdo`，与上述相同，但 `pdo_mappings: ['rx']`
- 强制对象。这些在代码生成阶段添加，值从 UI 控件填充。

代码生成将所有值复制到单个 OD 中，添加 PDO 映射和 SM 分配。

## PDO 映射

目前仅支持单个、非动态的 PDO，分别用于 TX 和 RX。

## 在 Bash 中可视化比较二进制文件（也适用于 git bash）

```bash
diff <(xxd et1100.bin) <(xxd lan9252.bin)
```

## Windows 的二进制文件比较工具：[VBinDiff](https://www.cjmweb.net/vbindiff/VBinDiff-Win32)

```cmd
VBinDiff et1100.bin  lan9252.bin
```

# 免责声明

EtherCAT 技术、商标名称和标志 "EtherCAT" 是 Beckhoff Automation GmbH 的知识产权，并受保护。
