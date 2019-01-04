# AAPT2出现问题


```java
Caused by: com.android.ide.common.process.ProcessException: Failed to execute aapt
        at com.android.builder.core.AndroidBuilder.processResources(AndroidBuilder.java:809)
        at com.android.builder.core.AndroidBuilder.processResources(AndroidBuilder.java:797)
        at com.android.build.gradle.internal.res.LinkApplicationAndroidResourcesTask.invokeAaptForSplit(LinkApplicationAndroidResourcesTask.java:491)
        ... 42 more
Caused by: java.util.concurrent.ExecutionException: java.util.concurrent.ExecutionException: com.android.builder.internal.aapt.v2.Aapt2Exception: AAPT2 error: check logs for details
        at com.google.common.util.concurrent.AbstractFuture.getDoneValue(AbstractFuture.java:503)
        at com.google.common.util.concurrent.AbstractFuture.get(AbstractFuture.java:482)
        at com.google.common.util.concurrent.AbstractFuture$TrustedFuture.get(AbstractFuture.java:79)
        at com.android.builder.internal.aapt.AbstractAapt.link(AbstractAapt.java:34)
        at com.android.builder.core.AndroidBuilder.processResources(AndroidBuilder.java:807)
        ... 44 more
Caused by: java.util.concurrent.ExecutionException: com.android.builder.internal.aapt.v2.Aapt2Exception: AAPT2 error: check logs for details
        at com.google.common.util.concurrent.AbstractFuture.getDoneValue(AbstractFuture.java:503)
        at com.google.common.util.concurrent.AbstractFuture.get(AbstractFuture.java:462)
        at com.google.common.util.concurrent.AbstractFuture$TrustedFuture.get(AbstractFuture.java:79)
        at com.android.builder.internal.aapt.v2.QueueableAapt2.lambda$makeValidatedPackage$1(QueueableAapt2.java:166)
Caused by: com.android.builder.internal.aapt.v2.Aapt2Exception: AAPT2 error: check logs for details
        at com.android.builder.png.AaptProcess$NotifierProcessOutput.handleOutput(AaptProcess.java:443)
        at com.android.builder.png.AaptProcess$NotifierProcessOutput.err(AaptProcess.java:395)
        at com.android.builder.png.AaptProcess$ProcessOutputFacade.err(AaptProcess.java:312)
        at com.android.utils.GrabProcessOutput$1.run(GrabProcessOutput.java:104)

```




