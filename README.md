# find_files.py
import argparse
import os
import sys

def find_files(root_dir, extensions):
    """
    Belirtilen klasörde (ve alt klasörlerinde) belirli uzantılara sahip dosyaları bulur.

    Args:
        root_dir (str): Aranacak kök klasör yolu.
        extensions (list): Aranacak dosya uzantılarının bir listesi (örn. ['.txt', '.py']).

    Returns:
        list: Bulunan dosyaların tam yollarının bir listesi.
    """
    found_files = []
    
    # Uzantıları küçük harfe çevirip bir küme (set) oluşturmak daha hızlı arama sağlar
    ext_set = {ext.lower() for ext in extensions}

    for dirpath, dirnames, filenames in os.walk(root_dir):
        # Alt klasörlerdeki tüm dosyaları kontrol et
        for filename in filenames:
            # Dosya adını ve uzantısını ayır
            _, file_extension = os.path.splitext(filename)
            
            # Uzantıyı küçük harfe çevirerek aranan uzantılar kümesinde olup olmadığını kontrol et
            if file_extension.lower() in ext_set:
                full_path = os.path.join(dirpath, filename)
                found_files.append(full_path)
    
    return found_files

def main():
    """
    Komut satırı argümanlarını işler ve dosya bulma işlevini çalıştırır.
    """
    parser = argparse.ArgumentParser(
        description="Belirtilen klasörde belirli uzantılara sahip dosyaları özyinelemeli olarak bulur ve listeler."
    )
    
    # Kök klasör argümanı (zorunlu)
    parser.add_argument(
        'directory',
        type=str,
        help="Aramanın yapılacağı kök klasörün yolu."
    )
    
    # Uzantılar argümanı (opsiyonel, varsayılan değer atanmış)
    parser.add_argument(
        '-e', '--ext',
        nargs='+',  # Birden fazla uzantı kabul et
        default=['.py', '.txt', '.md'], # Varsayılan olarak aranacak uzantılar
        help="Aranacak dosya uzantıları (boşlukla ayrılmış). Örn: -e .py .txt"
    )

    args = parser.parse_args()

    # Kök klasörün var olup olmadığını kontrol et
    if not os.path.isdir(args.directory):
        print(f"Hata: Belirtilen klasör yolu bulunamadı veya bir klasör değil: {args.directory}", file=sys.stderr)
        sys.exit(1)

    print(f"🔎 Klasör: **{args.directory}** içinde uzantıları: **{', '.join(args.ext)}** olan dosyalar aranıyor...\n")

    try:
        results = find_files(args.directory, args.ext)
        
        if results:
            print(f"🎉 **{len(results)}** dosya bulundu:")
            print("-" * 30)
            for file_path in results:
                print(f"| {file_path}")
            print("-" * 30)
        else:
            print("🙁 Belirtilen kriterlere uygun dosya bulunamadı.")

    except Exception as e:
        print(f"Beklenmeyen bir hata oluştu: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == '__main__':
    main()
