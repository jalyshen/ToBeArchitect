# 自定义分词器

***原创***



Lucene提供的默认分词器的功能都是最基础的，往往不能适用于真实的项目中，尤其是电商平台。

幸好，Lucene提供了自定义分词器的拓展，我们可以根据自己的需要来定制分词器。

Lucene版本采用8.2。

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

这个类封装了TokenStream，而TokenStream是Lucene的分词处理器。Lucene的Tokenizer继承与TokenStream。



根据上面的代码，Lucene提供客户自定义分词器的模式大致如下：

1. 通过overwrite Analyzer的createComponents(String var1)，构造一个TokenStreamComponents实例。而要实例化一个TokenStreamComponents实例，就需要一个TokenStream实例。
2. 显然，Analyzer并不是真正编写分词器的地方，而是TokenStream。
3. TokenStreamComponents就是粘和Analyzer和TokenStream的和事佬了。
4. 编写自定义的分词器TokenStream（具体而言，就是TokenStream的子类Tokenizer），然后在第一步处实例化，就完成了自定义分词器与Lucene的结合了。

通过一个Lucene内置的分词器 *EnglishAnalyzer*，来证实上面的想法：

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

从这个EnglishAnalyzer来看，一个分词器，由一个Tokenizer，0个或多个xxxFiler (TokenStream) “串连”而成。这个类似于Servlet的Filter机制。



## 编写自己的分词器

这里分析一个大家常用的分词器elasticsearch-analysis-ik，看看编写一个Lucene的分词器大致需要如何做。

### 第一步

创建 *org.apache.lucene.analysis.Analyzer* 的子类，把自定义的分词器引入Lucene体系。 IK的Analysis如下：

```java
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.Tokenizer;
import org.wltea.analyzer.cfg.Configuration;

/**
 * IK分词器，Lucene Analyzer接口实现
 * 兼容Lucene 4.0版本
 */
public final class IKAnalyzer extends Analyzer{
	
	private Configuration configuration;

	/**
	 * IK分词器Lucene  Analyzer接口实现类
	 * 
	 * 默认细粒度切分算法
	 */
	public IKAnalyzer(){
	}

    /**
	 * IK分词器Lucene Analyzer接口实现类
	 * 
	 * @param configuration IK配置
	 */
	public IKAnalyzer(Configuration configuration){
		  super();
          this.configuration = configuration;
	}


	/**
	 * 重载Analyzer接口，构造分词组件
	 */
	@Override
	protected TokenStreamComponents createComponents(String fieldName) {
        Tokenizer _IKTokenizer = new IKTokenizer(configuration);
	    return new TokenStreamComponents(_IKTokenizer);
  }

}
```

核心的代码，就是最后一个方法，重写 createComponents(String fieldName)。这个方法里，把自定义的Tokenizer引入Lucene体系。



### 第二步

第一步，在IKAnalysis里引入了IKTokenizer。显然，这就是第二步要做的事儿：创建自己的Tokenizer。

自定义的Tokenizer需要继承Lucene的Tokenizer，而Tokenizer又拓展于TokenStream。

先来看看TokenStream的核心方法：

```java
public abstract class TokenStream extends AttributeSource implements Closeable {
    .... // 省略一些代码
    
    // 需要子类实现
    public abstract boolean incrementToken() throws IOException;
    
    .... // 省略一些代码
}
```

从这个抽象方法可以明白，一个Token，就是一个词元，而 *incrementToken()* 就是从文本中获取一个***有意义的词元***。 即：分词逻辑应该由此方法实现。

再看Tokenizer，没有实现该方法。这个方法的实现，留给具体的分词逻辑处理。终于，来看看IK分词器的IKTokenizer。 具体如何做，都写在了代码的注释里。

```java
public final class IKTokenizer extends Tokenizer {

	...... // 省略一些代码
	
    /** 
	 * 从Lucene的角度看，这个方法，就是真实的分词逻辑所在。
	 * 
	 * 但是，如果我们自己写，可以再OO化。
	 * IK就是委托给了_IKImplement (IKSegmenter类的一个实例)
	 * 
	 */
	@Override
	public boolean incrementToken() throws IOException {
		//清除所有的词元属性
		clearAttributes();
        skippedPositions = 0;

        // ======================
        // Lexeme就是IK定义的词元对象。
        // 这里实现了词元的分词逻辑。
        // ======================
        Lexeme nextLexeme = _IKImplement.next();
    
		if(nextLexeme != null){
            posIncrAtt.setPositionIncrement(skippedPositions +1 );

			//将Lexeme转成Attributes
			//设置词元文本
			termAtt.append(nextLexeme.getLexemeText());
			//设置词元长度
			termAtt.setLength(nextLexeme.getLength());
			//设置词元位移
            offsetAtt.setOffset(correctOffset(
                      nextLexeme.getBeginPosition()), 
                      correctOffset(nextLexeme.getEndPosition()));

            //记录分词的最后位置
			endPosition = nextLexeme.getEndPosition();
			//记录词元分类
			typeAtt.setType(nextLexeme.getLexemeTypeString());			
			//返会true告知还有下个词元
			return true;
		}
		//返会false告知词元输出完毕
		return false;
	}
  
    /**
	 * 这个是org.apache.lucene.analysis.Tokenizer的方法
	 * 一个词元分出来后，需要重置一下
	 */
	@Override
	public void reset() throws IOException {
		super.reset();             // 父类中设置了新的输入内容
		_IKImplement.reset(input); // 设置新的内容，用于下一次分词
        skippedPositions = 0;
	}	
	
	...... // 省略一些代码
}
```



### 总结

编写一个Lucene的分词器还是很容易的。要实现的方法也不多。