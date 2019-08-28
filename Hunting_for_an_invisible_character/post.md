Recently, I spent some time trying to understand how an xml sent from a server to another become corrupted. 
I investigated the xml to be sent and it was all right, however, as soon as it was received the system produced an exception. 
I finally saw that a question mark was prefixing the xml.

It was obvious that this was an encoding problem, however, at first, I couldn't replicate it.

So, try to change the encoding of this page to ISO-8859-1 or another equivalent standard and see what happens on the following line.

```
this is a simple text﻿
```

Now, this page has several characters that are supported by UTF-8 and aren't supported by ISO-8859. For example this character ', is a single quote in UTF-8, and something else in ISO-8859.

The problem begins with the first example. This is a simple text, is a sequence of characters that have equivalent values across western standards. However, when we change the encoding something appears in the end of the phrase. What is that?

It's easy to understand that this must be a UTF-8 non printable character that has no equivalent on other encodings. And yet, which character will not appear with UTF-8.

Again, it's not a difficult case to solve when you understand it. After some searching in my prefered search engine I came across this unicode character [U+FEFF](https://www.fileformat.info/info/unicode/char/feff/index.htm). It's name is Unicode Character 'ZERO WIDTH NO-BREAK SPACE'. Amazing!!!

Reading more we can find out that this character is to be used as a byte-order mark or BOM. Read more about BOM [here](https://www.w3.org/International/questions/qa-byte-order-mark).

The Java code for replicating this effect is as follows:
```java
public static void main(String[] args) throws Exception {
    String test = "this a simple text";
    String string = test + '\uFEFF';

    OutputStream b1 = new ByteArrayOutputStream();
    OutputStream b2 = new ByteArrayOutputStream();
    StreamUtils.copy(string, StandardCharsets.ISO_8859_1, b1);
    StreamUtils.copy(string, StandardCharsets.UTF_8, b2);

    System.out.print("Is ");
    System.out.print(b1.toString());
    System.out.print(" or is ");
    System.out.print(b2.toString());
}
``` 

That prints the following line: 
```
Is this a simple text? or is this a simple text
```

I find it funny how the ISO-8859-1 prints this character as a question mark, changing, in this example, the meaning of the phrase.
