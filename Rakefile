require 'fileutils'
require 'rake/clean'
require 'yaml'

BOOK = "book"
BOOK_PDF = BOOK+".pdf"
BOOK_EPUB = BOOK+".epub"
CATALOG_FILE = "catalog.yml"
CONFIG_FILE = "config.yml"
WEBROOT = "webroot"
STYLE_FILE = "style.css"
PAGE_LIST_FILE = "page_list.md"

# キャッシュを削除
# sh "rm *.epub *.re"

# = reviewのファイル出力処理
# == Parameters:
# mode::
#  引数の指定として
#    'html','xml,'tex'
# chapter::
#  引数の指定として
#    ENV['re']
def build(mode, chapter)
  sh "review-compile --target=#{mode} --footnotetext --stylesheet=#{STYLE_FILE} #{chapter} > tmp"
  mode_ext = {"html" => "html", "latex" => "tex",
              "idgxml" => "xml"}
  FileUtils.mv "tmp", chapter.gsub(/re\z/, mode_ext[mode])
end

# = reviewのファイル出力処理
# == Parameters:
# mode::
#  引数の指定として
#    'html','xml,'tex'
def build_all(mode)
  sh "review-compile --target=#{mode} --footnotetext --stylesheet=#{STYLE_FILE}"
end

# = ページリストに記載されているmdのreview変換処理
def convert_page_list
  catalog = Hash.new { |h, k| h[k] = [] }
  File.read("#{PAGE_LIST_FILE}").scan(/\((.*.md)/).flatten.each do |file|
    case file
    when /predef/
      # predefが含まれるmdファイル名は前付
      catalog['PREDEF'] << file.ext('.re')
    when /chaps/
      # chapsが含まれるmdファイル名は章
      catalog['CHAPS'] << file.ext('.re')
    when /appendix/
      # appendixが含まれるmdファイル名は付録
      catalog['APPENDIX'] << file.ext('.re')
    when /postdef/
      # postdefが含まれるmdファイル名は後付
      catalog['POSTDEF'] << file.ext('.re')
    end
  end
  # catalog.ymlにビルド対象のreviewファイル名を記載
  File.write(CATALOG_FILE, YAML.dump(catalog))
end

# rake
# オプション無しでrake叩くときはrake pdfと同様
task :default => :pdf

# rake html
desc "build html (Usage: rake build re=target.re)"
task :html do
  if ENV['re'].nil?
    puts "Usage: rake build re=target.re"
    exit
  end
  build("html", ENV['re'])
end

# rake html_all
desc "build all html"
task :html_all do
  build_all("html")
end

# rake preproc
desc 'preproc all'
task :preproc do
  Dir.glob("*.re").each do |file|
    sh "review-preproc --replace #{file}"
  end
end

# rake all
# pdfとepubの出力を行う
desc 'generate PDF and EPUB file'
task :all => [:pdf, :epub]

# rake pdf
# コマンド実行されたときにPDF出力する
desc 'generate PDF file'
task :pdf => BOOK_PDF

# rake web
# コマンド実行されたときにHTML出力する
desc 'generate stagic HTML file for web'
task :web => WEBROOT

# rake epub
# コマンド実行されたときにEPUB出力する
desc 'generate EPUB file'
task :epub => BOOK_EPUB

SRC = FileList['*.md'] + %w(page_list.md)
OBJ = SRC.ext('re') + [CATALOG_FILE]
INPUT = OBJ + [CONFIG_FILE]


rule '.re' => '.md' do |t|
  # フォルダにあるmdファイルをreviewに変換するか見る
  case t.source
    when 'readme.md'
      # このファイルはビルド対象外
    when PAGE_LIST_FILE
      # このファイルはビルド対象外
    else
      # mdファイルをreファイルに変換する
      sh "bundle exec md2review --render-link-in-footnote #{t.source} > #{t.name}"
    end
end

file CATALOG_FILE => PAGE_LIST_FILE do |t|
  # ページリストがあるときに変換処理を行う
  convert_page_list
end

file BOOK_PDF => INPUT do
  FileUtils.rm_rf [BOOK_PDF, BOOK, BOOK+"-pdf"]
  # PDFのビルド
  sh "review-pdfmaker #{CONFIG_FILE}"
end

file BOOK_EPUB => INPUT do
  FileUtils.rm_rf [BOOK_EPUB, BOOK, BOOK+"-epub"]
  # EPUBのビルド
  sh "review-epubmaker #{CONFIG_FILE}"
end

file WEBROOT => INPUT do
  FileUtils.rm_rf [WEBROOT]
  sh "review-webmaker #{CONFIG_FILE}"
end

CLEAN.include([OBJ, BOOK, BOOK_PDF, BOOK_EPUB, BOOK+"-pdf", BOOK+"-epub", WEBROOT])
