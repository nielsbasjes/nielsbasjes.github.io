---
layout: post
title: Writing Avro to Parquet
date: '2015-12-14T14:33:00.001+01:00'
author: Niels Basjes
tags:
- Avro
- Java Generics
- Parquet
modified_time: '2015-12-14T14:35:10.457+01:00'
---

Today I tried to write records created using Apache Avro to disk using Apache Parquet in Java.
Although this seems like a trivial task I ran into a big obstacle which was primarily caused by a lack in my knowledge of how generics in Java work.

Because all constructors are either deprecated or package private the builder pattern is the way to go.
So I simply tried this (assisted by the fact that IntelliJ 'suggested' this):

{% highlight java %}
Path outputPath = new Path("/home/nbasjes/tmp/measurement.parquet");
ParquetWriter<Measurement> parquetWriter =
    AvroParquetWriter<Measurement>.builder(outputPath)
        .withSchema(Measurement.getClassSchema())
        .withCompressionCodec(CompressionCodecName.GZIP)
        .build();
{% endhighlight %}

This fails to compile with terrible errors and does not yield an understandable reason why it does so.None of the unit tests in the Parquet code base use the builder pattern yet; they are still on the methods that have been deprecated.

Looking at the source of the class that needs to do the work you'll see this:

{% highlight java %}
public class AvroParquetWriter<T> extends ParquetWriter<T> {
  public static <T> Builder<T> builder(Path file) {
    return new Builder<T>(file);
  }
{% endhighlight %}

After talking to a colleague of mine he pointed out that calling a static method in generic has a slightly different syntax.
Apparently calling such a method requires the following syntax (which works like a charm):

{% highlight java %}
Path outputPath = new Path("/home/nbasjes/tmp/measurement.parquet");
ParquetWriter<Measurement> parquetWriter =
   AvroParquetWriter.<Measurement>builder(outputPath)
       .withSchema(Measurement.getClassSchema())
       .withCompressionCodec(CompressionCodecName.GZIP)
       .build();
{% endhighlight %}

Let me guess; You don't see the difference?
It is the '.' before the call to the static builder method.

So instead of this

{% highlight java %}
AvroParquetWriter<Measurement>.builder(outputPath)
{% endhighlight %}

you must do this

{% highlight java %}
AvroParquetWriter.<Measurement>builder(outputPath)
{% endhighlight %}
