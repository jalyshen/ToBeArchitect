# 自定义分词器

***原创***



Lucene提供的默认分词器的功能都是最基础的，往往不能适用于真实的项目中，尤其是电商平台。

幸好，Lucene提供了自定义分词器的拓展，我们可以根据自己的需要来定制分词器。

## Lucene提供的入口

Lucene提供了一个抽象类 *org.apache.lucene.analysis.Analyzer* ，它主要有两个final的方法，这两个方法都返回TokenStream：

```java
public final TokenStream tokenStream(String fieldName, Reader reader) {
        Analyzer.TokenStreamComponents components = 
                      this.reuseStrategy.getReusableComponents(this, fieldName);
        Reader r = this.initReader(fieldName, reader);
        if (components == null) {
            components = this.createComponents(fieldName);
            this.reuseStrategy.setReusableComponents(this, fieldName, components);
        }

        components.setReader(r);
        return components.getTokenStream();
    }

public final TokenStream tokenStream(String fieldName, String text) {
        Analyzer.TokenStreamComponents components =    
                        this.reuseStrategy.getReusableComponents(this, fieldName);
        ReusableStringReader strReader = components != null && 
                components.reusableStringReader != null ? 
                components.reusableStringReader : new ReusableStringReader();
        strReader.setValue(text);
        Reader r = this.initReader(fieldName, strReader);
        if (components == null) {
            components = this.createComponents(fieldName);
            this.reuseStrategy.setReusableComponents(this, fieldName, components);
        }

        components.setReader(r);
        components.reusableStringReader = strReader;
        return components.getTokenStream();
    }
```

这两个方法的逻辑基本一样： 

如果能从reuseStrategy中获取components，就把要分析的内容（Reader对象，或者String-也要转化为reader）交给此components来处理。如果Lucene启动时，没有初始化一些components，则在此时创建出来。

所以，这两个方法中，核心的一行是：

```java
components = this.createComponents(fieldName);
```

再看看Analyzer的createComponents方法：

```java
protected abstract Analyzer.TokenStreamComponents createComponents(String var1);
```

嚯，**居然是抽象方法，就是提供我们自定义分析器的入口了**。



再来看看这个TokenStreamComponents：

```java
public static final class TokenStreamComponents {
        protected final Consumer<Reader> source;
        protected final TokenStream sink;
        transient ReusableStringReader reusableStringReader;

        public TokenStreamComponents(Consumer<Reader> source, TokenStream result) {
            this.source = source;
            this.sink = result;
        }

        public TokenStreamComponents(Tokenizer tokenizer, TokenStream result) {
            this(tokenizer::setReader, result);
        }

        public TokenStreamComponents(Tokenizer tokenizer) {
            this((Consumer)(tokenizer::setReader), tokenizer);
        }

        private void setReader(Reader reader) {
            this.source.accept(reader);
        }

        public TokenStream getTokenStream() {
            return this.sink;
        }

        public Consumer<Reader> getSource() {
            return this.source;
        }
    }
```

这个类封装了TokenStream，而TokenStream是Lucene的分词处理器。



根据上面的代码，Lucene提供客户自定义分词器的模式大致如夏：

1. 通过overwrite Analyzer的createComponents(String var1)，构造一个TokenStreamComponents实例。而要实例化一个TokenStreamComponents实例，就需要一个TokenStream实例。
2. 显然，Analyzer并不是真正编写分词器的地方，而是TokenStream。
3. TokenStreamComponents就是粘和Analyzer和TokenStream的和事佬了。
4. 编写自定义的分词器TokenStream（具体而言，就是TokenStream的子类Tokenizer），然后在第一步处实例化，就完成了自定义分词器与Lucene的结合了。

我们来看一个Lucene内置的分词器 *EnglishAnalyzer*，来证实上面的想法：

```java
public final class EnglishAnalyzer extends StopwordAnalyzerBase {
    
    ...... // 省略非关键代码

    protected TokenStreamComponents createComponents(String fieldName) {
        // 实例化一个分词器。Tokenizer也是一个TokenStream
        Tokenizer source = new StandardTokenizer(); 
      
        // 下面实例化一堆过滤器。 XXXFilter也是TokenStream的子类
        TokenStream result = new EnglishPossessiveFilter(source); 
        TokenStream result = new LowerCaseFilter(result); 
        TokenStream result = new StopFilter(result, this.stopwords);
        if (!this.stemExclusionSet.isEmpty()) {
            result = new SetKeywordMarkerFilter((TokenStream)result, 
                                                this.stemExclusionSet);
        }

        TokenStream result = new PorterStemFilter((TokenStream)result);
        return new TokenStreamComponents(source, result);
    }

    ...... // 省略非关键代码
}
```



## 编写自定义分词器

