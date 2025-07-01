## 处理目标文件夹下大量txt文本中的ai提示词
```
import os
import re
from glob import glob

def clean_text(text):
    # 定义要在第一个<p>标签中删除的模式
    first_p_patterns = [
        r'随着',               # 随着
        r'随着.*?发展，',      # 随着...发展，
        r'在.*?上，',          # 在...上，
        r'在.*?中，',          # 在...中，
        r'在.*?下，',          # 在...下，
        r'随着.*?发展',        # 随着...发展
        r'在.*?上',            # 在...上
        r'在.*?中',            # 在...中
        r'在.*?下',            # 在...下
    ]
    
    # 定义要在全文中删除的模式
    global_patterns = [
        r'最近，',             # 最近，
        r'首先，',             # 首先，
        r'其次，',             # 其次，
        r'然后，',             # 然后，
        r'然而，',             # 然而，
        r'总之，',             # 总之，
        r'此外，',             # 此外，
        r'同时，',             # 同时，
        r'总的来说，',         # 总的来说，
        r'综上所述，',         # 综上所述，
        r'总体而言，',         # 总体而言，

        r'最近',              # 最近
        r'首先',              # 首先
        r'其次',              # 其次
        r'然后',              # 然后
        r'然而',              # 然而
        r'总之',              # 总之
        r'此外',              # 此外
        r'同时',              # 同时
        r'总的来说',          # 总的来说
        r'综上所述',          # 综上所述
        r'总体而言',          # 总体而言

        r'总结：',          # 总结：
        r'结论：',          # 结论：
        r'结语：',          # 结语：

        r'<[^>]*>[一二三四五六七八九十]+、总结</[^>]*>', # <任意html标签>X、总结</任意html标签>
        r'<[^>]*>总结</[^>]*>', # <任意html标签>总结</任意html标签>
        r'<[^>]*>[一二三四五六七八九十]+、结语</[^>]*>', # <任意html标签>X、结语</任意html标签>
        r'<[^>]*>结语</[^>]*>', # <任意html标签>结语</任何html标签>
        r'<[^>]*>[一二三四五六七八九十]+、结论</[^>]*>', # <任意html标签>X、结论</任意html标签>
        r'<[^>]*>结论</[^>]*>', # <任意html标签>结论</任何html标签>
    ]
    
    # 处理第一个<p>标签
    def process_first_p(match):
        p_attrs = match.group(1)  # 捕获<p>标签的属性
        p_content = match.group(2)  # 捕获<p>标签的内容
        
        # 在内容中删除特定的模式
        for pattern in first_p_patterns:
            p_content = re.sub(pattern, '', p_content)
        
        # 返回处理后的<p>标签，保留原有属性和未删除的内容
        return f'<p{p_attrs}>{p_content}</p>'
    
    # 只在第一个<p>标签中应用特定模式
    text = re.sub(r'<p([^>]*)>(.*?)</p>', process_first_p, text, count=1, flags=re.DOTALL)
    
    # 在全文中应用其他模式
    for pattern in global_patterns:
        text = re.sub(pattern, '', text)
    
    return text

def process_files(input_dir, output_dir):
    # 确保输出目录存在
    os.makedirs(output_dir, exist_ok=True)
    
    # 获取所有txt文件
    txt_files = glob(os.path.join(input_dir, '**/*.txt'), recursive=True)
    
    for file_path in txt_files:
        # 读取文件内容
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()
        
        # 清理文本
        cleaned_content = clean_text(content)
        
        # 构建输出路径
        rel_path = os.path.relpath(file_path, input_dir)
        out_path = os.path.join(output_dir, rel_path)
        
        # 确保输出子目录存在
        os.makedirs(os.path.dirname(out_path), exist_ok=True)
        
        # 写入清理后的内容
        with open(out_path, 'w', encoding='utf-8') as f:
            f.write(cleaned_content)
        
        print(f'Processed: {file_path} -> {out_path}')

if __name__ == '__main__':
    input_directory = input('请输入目标文件夹路径: ')
    output_directory = os.path.join(input_directory, 'cleaned')
    
    process_files(input_directory, output_directory)
    print('处理完成！清理后的文件保存在:', output_directory)

```
