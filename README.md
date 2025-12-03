# Cloudfare-SpeesTest-Preferred-IP-format-conversion
Cloudfare SpeesTest优选出来的IP快速转换成CFnew上传API的自动脚本

脚本逻辑：
自动查找 result.csv 或 result.txt。
自动识别 IPv4/IPv6。
自动处理重名编号（SJC1, SJC2）。
强制在行尾添加英文逗号 ,。
最终完整版 Python 脚本 (convert_final_v2.py)
code
Python  蟒
import re
import os
import sys

# ================= 配置区域 =================
# 输出文件名
OUTPUT_FILE = "yxip.txt"

# 地区三字码映射表 (国旗 + 国家 + 城市)
COLO_MAP = {
    # --- 亚太地区 - 中国及周边 ---
    "HKG": "🇭🇰中国香港",
    "TPE": "🇹🇼中国台湾台北",

    # --- 亚太地区 - 日本 ---
    "NRT": "🇯🇵日本东京成田",
    "KIX": "🇯🇵日本大阪",
    "ITM": "🇯🇵日本大阪伊丹",
    "FUK": "🇯🇵日本福冈",

    # --- 亚太地区 - 韩国 ---
    "ICN": "🇰🇷韩国首尔仁川",

    # --- 亚太地区 - 东南亚 ---
    "SIN": "🇸🇬新加坡",
    "BKK": "🇹🇭泰国曼谷",
    "HAN": "🇻🇳越南河内",
    "SGN": "🇻🇳越南胡志明市",
    "MNL": "🇵🇭菲律宾马尼拉",
    "CGK": "🇮🇩印度尼西亚雅加达",
    "KUL": "🇲🇾马来西亚吉隆坡",
    "RGN": "🇲🇲缅甸仰光",
    "PNH": "🇰🇭柬埔寨金边",

    # --- 亚太地区 - 南亚 ---
    "BOM": "🇮🇳印度孟买",
    "DEL": "🇮🇳印度新德里",
    "MAA": "🇮🇳印度金奈",
    "BLR": "🇮🇳印度班加罗尔",
    "HYD": "🇮🇳印度海得拉巴",
    "CCU": "🇮🇳印度加尔各答",

    # --- 亚太地区 - 澳洲/新西兰 ---
    "SYD": "🇦🇺澳大利亚悉尼",
    "MEL": "🇦🇺澳大利亚墨尔本",
    "BNE": "🇦🇺澳大利亚布里斯班",
    "PER": "🇦🇺澳大利亚珀斯",
    "AKL": "🇳🇿新西兰奥克兰",

    # --- 北美地区 - 美国西海岸 ---
    "LAX": "🇺🇸美国洛杉矶",
    "SJC": "🇺🇸美国圣何塞",
    "SEA": "🇺🇸美国西雅图",
    "SFO": "🇺🇸美国旧金山",
    "PDX": "🇺🇸美国波特兰",
    "SAN": "🇺🇸美国圣地亚哥",
    "PHX": "🇺🇸美国凤凰城",
    "LAS": "🇺🇸美国拉斯维加斯",

    # --- 北美地区 - 美国东海岸 ---
    "EWR": "🇺🇸美国纽瓦克",
    "IAD": "🇺🇸美国华盛顿",
    "BOS": "🇺🇸美国波士顿",
    "PHL": "🇺🇸美国费城",
    "ATL": "🇺🇸美国亚特兰大",
    "MIA": "🇺🇸美国迈阿密",
    "MCO": "🇺🇸美国奥兰多",

    # --- 北美地区 - 美国中部 ---
    "ORD": "🇺🇸美国芝加哥",
    "DFW": "🇺🇸美国达拉斯",
    "IAH": "🇺🇸美国休斯顿",
    "DEN": "🇺🇸美国丹佛",
    "MSP": "🇺🇸美国明尼阿波利斯",
    "DTW": "🇺🇸美国底特律",
    "STL": "🇺🇸美国圣路易斯",
    "MCI": "🇺🇸美国堪萨斯城",

    # --- 北美地区 - 加拿大 ---
    "YYZ": "🇨🇦加拿大多伦多",
    "YVR": "🇨🇦加拿大温哥华",
    "YUL": "🇨🇦加拿大蒙特利尔",

    # --- 欧洲地区 - 西欧 ---
    "LHR": "🇬🇧英国伦敦",
    "CDG": "🇫🇷法国巴黎",
    "FRA": "🇩🇪德国法兰克福",
    "AMS": "🇳🇱荷兰阿姆斯特丹",
    "BRU": "🇧🇪比利时布鲁塞尔",
    "ZRH": "🇨🇭瑞士苏黎世",
    "VIE": "🇦🇹奥地利维也纳",
    "MUC": "🇩🇪德国慕尼黑",
    "DUS": "🇩🇪德国杜塞尔多夫",
    "HAM": "🇩🇪德国汉堡",

    # --- 欧洲地区 - 南欧 ---
    "MAD": "🇪🇸西班牙马德里",
    "BCN": "🇪🇸西班牙巴塞罗那",
    "MXP": "🇮🇹意大利米兰",
    "FCO": "🇮🇹意大利罗马",
    "ATH": "🇬🇷希腊雅典",
    "LIS": "🇵🇹葡萄牙里斯本",

    # --- 欧洲地区 - 北欧 ---
    "ARN": "🇸🇪瑞典斯德哥尔摩",
    "CPH": "🇩🇰丹麦哥本哈根",
    "OSL": "🇳🇴挪威奥斯陆",
    "HEL": "🇫🇮芬兰赫尔辛基",

    # --- 欧洲地区 - 东欧 ---
    "WAW": "🇵🇱波兰华沙",
    "PRG": "🇨🇿捷克布拉格",
    "BUD": "🇭🇺匈牙利布达佩斯",
    "OTP": "🇷🇴罗马尼亚布加勒斯特",
    "SOF": "🇧🇬保加利亚索非亚",

    # --- 中东地区 ---
    "DXB": "🇦🇪阿联酋迪拜",
    "TLV": "🇮🇱以色列特拉维夫",
    "BAH": "🇧🇭巴林",
    "AMM": "🇯🇴约旦安曼",
    "KWI": "🇰🇼科威特",
    "DOH": "🇶🇦卡塔尔多哈",
    "MCT": "🇴🇲阿曼马斯喀特",

    # --- 南美地区 ---
    "GRU": "🇧🇷巴西圣保罗",
    "GIG": "🇧🇷巴西里约热内卢",
    "EZE": "🇦🇷阿根廷布宜诺斯艾利斯",
    "BOG": "🇨🇴哥伦比亚波哥大",
    "LIM": "🇵🇪秘鲁利马",
    "SCL": "🇨🇱智利圣地亚哥",

    # --- 非洲地区 ---
    "JNB": "🇿🇦南非约翰内斯堡",
    "CPT": "🇿🇦南非开普敦",
    "CAI": "🇪🇬埃及开罗",
    "LOS": "🇳🇬尼日利亚拉各斯",
    "NBO": "🇰🇪肯尼亚内罗毕",
    "ACC": "🇬🇭加纳阿克拉",
    
    # --- 兜底 ---
    "UNKNOWN": "🏳️未知地区"
}
# ===========================================

def get_input_file():
    """智能查找输入文件"""
    candidates = ["result.csv", "result.txt"]
    for filename in candidates:
        if os.path.exists(filename):
            print(f"📂 自动发现输入文件: {filename}")
            return filename
    
    # 没找到则提示输入
    while True:
        print(f"❌ 未找到默认文件 {candidates}")
        user_input = input("请手动输入测速结果文件名: ").strip()
        if os.path.exists(user_input):
            return user_input
        else:
            print("❌ 文件不存在，请重新输入。")

def is_ipv6(ip):
    """判断是否为 IPv6 地址"""
    return ":" in ip

def process_conversion():
    input_file = get_input_file()
    
    # 计数器：用于记录每个地区出现的次数 (SJC1, SJC2...)
    remark_counters = {}
    results = []

    try:
        with open(input_file, 'r', encoding='utf-8') as f:
            lines = f.readlines()

        print(f"🔄 正在处理 {len(lines)} 行数据...")

        for line in lines:
            line = line.strip()
            # 跳过空行或标题行
            if not line or ("IP" in line and "速度" in line):
                continue

            # 使用正则按空白字符分割
            parts = re.split(r'\s+', line)

            # 确保行数据足够
            if len(parts) < 2:
                continue

            ip = parts[0]     # 第一列是 IP
            
            # 获取最后一列作为地区码
            colo = parts[-1]
            if len(colo) != 3 and len(parts) > 2:
                 # 尝试倒数第二列，防止有空列情况
                 if len(parts[-2]) == 3:
                     colo = parts[-2]

            # --- 生成备注逻辑 ---
            # 1. 获取中文映射名称 (例如: 🇺🇸美国圣何塞)
            remark_prefix = COLO_MAP.get(colo, f"🏳️{colo}")
            
            # 2. 拼接基础名称 (例如: 🇺🇸美国圣何塞SJC)
            base_remark = f"{remark_prefix}{colo}" 

            # 3. 计数逻辑 (SJC1, SJC2)
            if base_remark not in remark_counters:
                remark_counters[base_remark] = 1
            else:
                remark_counters[base_remark] += 1
            
            # 4. 生成最终备注 (例如: 🇺🇸美国圣何塞SJC1)
            final_remark = f"{base_remark}{remark_counters[base_remark]}"

            # --- 格式化 IP ---
            if is_ipv6(ip):
                formatted_ip = f"[{ip}]:443"
            else:
                formatted_ip = f"{ip}:443"

            # --- 拼接最终结果 (结尾加英文逗号) ---
            # 格式: [IP]:443#备注,
            final_line = f"{formatted_ip}#{final_remark},"
            results.append(final_line)

        # 写入结果
        if results:
            with open(OUTPUT_FILE, 'w', encoding='utf-8') as f:
                f.write('\n'.join(results))
            
            print("\n" + "="*30)
            print(f"✅ 转换成功！")
            print(f"📥 输入文件: {input_file}")
            print(f"📤 输出文件: {OUTPUT_FILE}")
            print(f"📊 有效节点: {len(results)} 个")
            print("="*30)
            
            # 预览
            print("\n👀 结果预览 (前3行):")
            for line in results[:3]:
                print(line)
        else:
            print("\n⚠️ 警告: 未提取到任何有效 IP，请检查输入文件格式。")

    except Exception as e:
        print(f"\n❌ 程序发生错误: {e}")

if __name__ == "__main__":

    process_conversion()
    
运行效果

假设您的输入文件包含 2606:.... SJC，运行后的输出文件 yxip.txt 内容将会是：

code

Text  发短信

[2606:4700:0:c378:df55:3ae4:394f:1e90]:443#🇺🇸美国圣何塞SJC1,

[2606:4700:1:1::1]:443#🇺🇸美国圣何塞SJC2,

1.1.1.1:443#🇭🇰中国香港HKG1,
以上是展示的内容
