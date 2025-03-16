---
layout: post
title:  Java Sound API – 捕获麦克风
category: java-os
copyright: java-os
excerpt: Java OS
---

## 1. 概述

在本文中，我们将了解如何使用Java捕获麦克风并录制传入的音频，并将其保存为WAV文件。为了捕获来自麦克风的传入声音，我们使用Java生态系统的一部分Java Sound API。

**Java Sound API是一个功能强大的API，用于捕获、处理和播放音频，由4个包组成。我们将重点介绍javax.sound.sampled包，它提供了捕获传入音频所需的所有接口和类**。

## 2. TargetDataLine是什么？

**[TargetDataLine](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/sound/sampled/TargetDataLine.html)是一种[DataLine](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/sound/sampled/DataLine.html)对象，我们使用它来捕获和读取与音频相关的数据，它从麦克风等音频捕获设备捕获数据**。该接口提供了读取和捕获数据所需的所有方法，它从目标数据线的缓冲区读取数据。

我们可以调用AudioSystem的getLine()方法并为其提供DataLine.Info对象，该对象提供了音频的所有传输控制方法，[Oracle文档](https://docs.oracle.com/javase/8/docs/technotes/guides/sound/programmer_guide/chapter2.html)详细解释了Java Sound API的工作原理。

让我们了解一下使用Java从麦克风捕获音频所需的步骤。

## 3. 捕捉声音的步骤

为了保存捕获的音频，Java支持：AU、AIFF、AIFC、SND和WAVE文件格式，我们将使用WAVE(.wav)文件格式来保存文件。

该过程的第一步是初始化AudioFormat实例，AudioFormat通知Java如何解释和处理传入声音流中的信息位，我们在示例中使用以下AudioFormat类构造函数：

```java
AudioFormat(AudioFormat.Encoding encoding, float sampleRate, int sampleSizeInBits, int channels, int frameSize, float frameRate, boolean bigEndian)
```

之后，我们打开一个DataLine.Info对象，此对象包含与数据线(输入)相关的所有信息。**使用DataLine.Info对象，我们可以创建TargetDataLine的一个实例，它将把所有传入的数据读入音频流**。为了生成TargetDataLine实例，我们使用AudioSystem.getLine()方法并传递DataLine.Info对象：

```java
line = (TargetDataLine) AudioSystem.getLine(info);
```

line是TargetDataLine实例，而info是DataLine.Info实例。

创建后，我们可以打开线路以读取所有传入的声音，**我们可以使用AudioInputStream来读取传入的数据。最后，我们可以将这些数据写入WAV文件并关闭所有流**。

为了理解这个过程，我们来看一下一个用于记录输入声音的小程序。

## 4. 示例应用程序

要查看Java Sound API的实际作用，让我们创建一个简单的程序。我们将把它分为三个部分，首先构建AudioFormat，其次构建TargetDataLine，最后将数据保存为文件。

### 4.1 构建AudioFormat

AudioFormat类定义了TargetDataLine实例可以捕获哪种数据，因此，第一步是在打开新的数据线之前初始化AudioFormat类实例。App类是应用程序的主类，负责进行所有调用。我们在名为ApplicationProperties的常量类中定义AudioFormat的属性，绕过所有必要的参数来构建AudioFormat实例：

```java
public static AudioFormat buildAudioFormatInstance() {
    ApplicationProperties aConstants = new ApplicationProperties();
    AudioFormat.Encoding encoding = aConstants.ENCODING;
    float rate = aConstants.RATE;
    int channels = aConstants.CHANNELS;
    int sampleSize = aConstants.SAMPLE_SIZE;
    boolean bigEndian = aConstants.BIG_ENDIAN;

    return new AudioFormat(encoding, rate, sampleSize, channels, (sampleSize / 8) * channels, rate, bigEndian);
}
```

现在我们已经准备好了AudioFormat，我们可以继续构建TargetDataLine实例。

### 4.2 构建TargetDataLine

我们使用TargetDataLine类从麦克风读取音频数据。在我们的示例中，我们在SoundRecorder类中获取并运行TargetDataLine。getTargetDataLineForRecord()方法构建TargetDataLine实例。

我们读取并处理音频输入并将其转储到AudioInputStream对象中，创建TargetDataLine实例的方式是：

```java
private TargetDataLine getTargetDataLineForRecord() {
    TargetDataLine line;
    DataLine.Info info = new DataLine.Info(TargetDataLine.class, format);
    if (!AudioSystem.isLineSupported(info)) {
        return null;
    }
    line = (TargetDataLine) AudioSystem.getLine(info);
    line.open(format, line.getBufferSize());
    return line;
}
```

### 4.3 构建和填充AudioInputStream

到目前为止，在我们的示例中，我们已经创建了一个AudioFormat实例并将其应用于TargetDataLine，并打开数据线以读取音频数据。我们还创建了一个线程来帮助自动运行SoundRecorder实例，我们首先在线程运行时构建一个字节输出流，然后将其转换为AudioInputStream实例。构建AudioInputStream实例所需的参数是：

```java
int frameSizeInBytes = format.getFrameSize();
int bufferLengthInFrames = line.getBufferSize() / 8;
final int bufferLengthInBytes = bufferLengthInFrames * frameSizeInBytes;
```

请注意，在上面的代码中，我们将bufferSize除了8，这样做是为了让缓冲区和数组的长度相同，以便记录器可以在读取数据后立即将其传送到线路。

现在我们已经初始化了所有需要的参数，下一步是构建字节输出流。下一步是将生成的输出流(捕获的声音数据)转换为AudioInputStream实例。

```java
buildByteOutputStream(out, line, frameSizeInBytes, bufferLengthInBytes);
this.audioInputStream = new AudioInputStream(line);

setAudioInputStream(convertToAudioIStream(out, frameSizeInBytes));
audioInputStream.reset();
```

在设置InputStream之前，我们将构建字节[OutputStream](https://www.baeldung.com/java-outputstream)：

```java
public void buildByteOutputStream(final ByteArrayOutputStream out, final TargetDataLine line, int frameSizeInBytes, final int bufferLengthInBytes) throws IOException {
    final byte[] data = new byte[bufferLengthInBytes];
    int numBytesRead;

    line.start();
    while (thread != null) {
        if ((numBytesRead = line.read(data, 0, bufferLengthInBytes)) == -1) {
            break;
        }
        out.write(data, 0, numBytesRead);
    }
}
```

然后我们[将字节Outstream转换为AudioInputStream](https://www.baeldung.com/convert-byte-array-to-input-stream)，如下所示：

```java
public AudioInputStream convertToAudioIStream(final ByteArrayOutputStream out, int frameSizeInBytes) {
    byte audioBytes[] = out.toByteArray();
    ByteArrayInputStream bais = new ByteArrayInputStream(audioBytes);
    AudioInputStream audioStream = new AudioInputStream(bais, format, audioBytes.length / frameSizeInBytes);
    long milliseconds = (long) ((audioInputStream.getFrameLength() * 1000) / format.getFrameRate());
    duration = milliseconds / 1000.0;
    return audioStream;
}
```

### 4.4 将AudioInputStream保存为Wav文件

我们已经创建并填充了AudioInputStream，并将其存储为SoundRecorder类的成员变量。我们将使用SoundRecorder实例getter属性在App类中检索此AudioInputStream，并将其传递给WaveDataUtil类：

```java
wd.saveToFile("/SoundClip", AudioFileFormat.Type.WAVE, soundRecorder.getAudioInputStream());
```

WaveDataUtil类包含将AudioInputStream转换为.wav文件的代码：

```java
AudioSystem.write(audioInputStream, fileType, myFile);
```

## 5. 总结

在本文中，我们介绍了使用Java捕捉麦克风声音的方法。