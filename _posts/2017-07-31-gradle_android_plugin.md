---
layout: post
title: Android Gradle Plugin资源合并逻辑分析
categories: [Gradle]
keywords: Gradle, plugin, android
---

Android Gradle插件处理资源时会将APP和Library的资源合并，如果资源重名，会用APP的资源覆盖Library的资源。

aapt编译资源时，如果资源重名，会报错

首先使用万能debug参数运行gradle编译：gradle (./gradlew) assembleDebug --debug，查看输出，看能否找到资源合并相关的日志输出，可惜的是没有找到直接相关的日志，但发现资源合并后存放的目录app/build/intermediates/res/merged/debug/被输出了好几次，那应该是资源合并处理逻辑输出的日志。

既然没找到直接相关的日志输出，还是使用merge关键字在源码目录下grep，果然找到了MergeResources这个类，但不熟悉Android Gradle的代码，也不好直接定位资源重名时的覆盖逻辑，所以在各个可疑的地方打下日志，同时为了弄清出调用关系，打日志的方式是直接通过new RuntimeException("message").printStacktrace()。

在protected void doFullTaskAction() throws IOException {方法上打印日志，然后在源码目录下执行./gradlew pL，编译出最新的Android Gradle Plugin，在APP端的gradle配置文件中，添加上一步导出的仓库，这样APP端就可以使用本地的编译的最新插件。

在APP端通过./gradlew assembleDebug编译APP，果然打出了相应日志，确实调用了doFullTaskAction方法。

一步一步看这个方法的实现

@Override

protected void doFullTaskAction() throws IOException {

​    new RuntimeException("MergeResources:doFullTaskAction").printStackTrace();

​    ResourcePreprocessor preprocessor = getPreprocessor();

​    // this is full run, clean the previous output

​    File destinationDir = getOutputDir();

​    FileUtils.cleanOutputDir(destinationDir);

​    List<ResourceSet> resourceSets = getConfiguredResourceSets(preprocessor);

​    // create a new merger and populate it with the sets.

​    ResourceMerger merger = new ResourceMerger(minSdk);

这个是一些基本的初始化，最需要注意的是ResourceMerger，

接下来merger第一步处理是添加各个resourceSet，那么resourcSet是什么东西了

try {

​    for (ResourceSet resourceSet : resourceSets) {

​        resourceSet.loadFromFiles(getILogger());

​        merger.addDataSet(resourceSet);

​    }

在ResourceMerger.addDataSet方法上添加日志看看

@Override

public void addDataSet(ResourceSet resourceSet) {

​    new RuntimeException("ResourceMerger:addDataSet:" + resourceSet).printStackTrace();

​    super.addDataSet(resourceSet);

}

打出来的日志如下：（省略了调用栈信息）

ResourceMerger:addDataSet:GeneratedResourceSet{24.2.1$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/support-compat/24.2.1/res]}

ResourceMerger:addDataSet:GeneratedResourceSet{24.2.1$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/support-core-ui/24.2.1/res]}

ResourceMerger:addDataSet:GeneratedResourceSet{24.2.1$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/recyclerview-v7/24.2.1/res]}

ResourceMerger:addDataSet:GeneratedResourceSet{0.1$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.miui.support/core/0.1/res]}

ResourceMerger:addDataSet:GeneratedResourceSet{main$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/src/main/res, /home/chenshun/AndroidStudioProjects/Testor/app/build/generated/res/rs/debug, /home/chenshun/AndroidStudioProjects/Testor/app/build/generated/res/resValues/debug]}

...

从日志看，ResourceSet就是一个资源目录res，包括引用库中的资源目录和APP中的资源目录res以及生成的中间资源目录，弄清楚了ResourceSet，接着往下看

// get the merged set and write it down.

Aapt aapt =

​        AaptGradleFactory.make(

​                getBuilder(),

​                getCrunchPng(),

​                getProcess9Patch(),

​                variantScope,

​                getAaptTempDir());

MergedResourceWriter writer = new MergedResourceWriter(

​        destinationDir,

​        getPublicFile(),

​        getBlameLogFolder(),

​        preprocessor,

​        aapt::compile,

​        getIncrementalFolder());

看来在合并资源时还借助了aapt对资源做一些处理，结合日志看应该是aapt crunch图片资源，暂时不用管它，往下看

merger.mergeData(writer, false /*doCleanUp*/);

// No exception? Write the known state.

merger.writeBlobTo(getIncrementalFolder(), writer, false);

这里应该是关键逻辑了，查看mergeData逻辑定义，基本没做什么处理，直接调用super.mergeData，看看定义，逻辑比较长，一点点看

public void mergeData(@NonNull MergeConsumer<I> consumer, boolean doCleanUp)

​        throws MergingException {

​    consumer.start(mFactory);

​    try {

​        // get all the items keys.

​        Set<String> dataItemKeys = Sets.newHashSet();

​        for (S dataSet : mDataSets) {

​            // quick check on duplicates in the resource set.

​            dataSet.checkItems();

​            ListMultimap<String, I> map = dataSet.getDataMap();

​            dataItemKeys.addAll(map.keySet());

​        }

consumer就是之前定义的MergedResourceWriter，第一步调用的start方法，查看定义什么都没做，先忽略。看看下面的checkItems，这个比较可疑，资源合并前先checkItem，check什么内容呢？看看里面的定义

protected void checkItems() throws DuplicateDataException {

​    System.err.println("checkItems:" + mItems);

​    if (!mValidateEnabled) {

​        return;

​    }

​    Collection<Collection<I>> duplicateCollections = Lists.newArrayList();

​    // check a list for duplicate, ignoring removed items.

​    for (Map.Entry<String, Collection<I>> entry : mItems.asMap().entrySet()) {

​        Collection<I> items = entry.getValue();

​        // there can be several version of the same key if some are "removed"

​        I lastItem = null;

​        for (I item : items) {

​            System.err.println("checkOneItems:" + item);

​            if (!item.isRemoved()) {

​                if (lastItem == null) {

​                    lastItem = item;

​                } else {

​                    // MIUI ADD: START

​                    // Because resource overlay is allowed, we just disable duplicate items

​                    // in the same file

​                    // NOTICE: only compare the last item because mItems is a list map,

​                    //         and the order is kept with items added to

​                    if (lastItem.getSource() != null

​                            && item.getSource() != null

​                            && lastItem.getSource().getFile() != null

​                            && !lastItem.getSource().getFile().equals(item.getSource().getFile())) {

​                        lastItem = item;

​                        continue;

​                    }

​                    // END

​                    // We have duplicates, store them and throw the exception later, so

​                    // the user gets all the error messages at once.

​                    duplicateCollections.add(items);

​                }

​            }

​        }

​    }

​    if (!duplicateCollections.isEmpty()) {

​        throw new DuplicateDataException(DuplicateDataException.createMessages(duplicateCollections));

​    }

}

从代码看，主要是检查资源是否有重复，居然有我们添加的逻辑，从注释来看好像是处理资源重名的，先注释看看会不会报错，注释后，跑起来还是没有报错，看看什么原因。加几个日志看看

checkItems:{attr/reverseLayout=[ResourceItem{mName='reverseLayout', mType=attr, mStatus=1}], attr/stackFromEnd=[ResourceItem{mName='stackFromEnd', mType=attr, mStatus=1}], dimen/item_touch_helper_swipe_escape_max_velocity=[ResourceItem{mName='item_touch_helper_swipe_escape_max_velocity', mType=dimen, mStatus=1}], attr/spanCount=[ResourceItem{mName='spanCount', mType=attr, mStatus=1}], id/item_touch_helper_previous_elevation=[ResourceItem{mName='item_touch_helper_previous_elevation', mType=id, mStatus=1}], declare-styleable/RecyclerView=[ResourceItem{mName='RecyclerView', mType=declare-styleable, mStatus=1}], attr/layoutManager=[ResourceItem{mName='layoutManager', mType=attr, mStatus=1}], dimen/item_touch_helper_swipe_escape_velocity=[ResourceItem{mName='item_touch_helper_swipe_escape_velocity', mType=dimen, mStatus=1}], dimen/item_touch_helper_max_drag_scroll_per_frame=[ResourceItem{mName='item_touch_helper_max_drag_scroll_per_frame', mType=dimen, mStatus=1}]}

checkOneItems:ResourceItem{mName='reverseLayout', mType=attr, mStatus=1}

checkOneItems:ResourceItem{mName='stackFromEnd', mType=attr, mStatus=1}

checkOneItems:ResourceItem{mName='item_touch_helper_swipe_escape_max_velocity', mType=dimen, mStatus=1}

checkOneItems:ResourceItem{mName='spanCount', mType=attr, mStatus=1}

checkOneItems:ResourceItem{mName='item_touch_helper_previous_elevation', mType=id, mStatus=1}

checkOneItems:ResourceItem{mName='RecyclerView', mType=declare-styleable, mStatus=1}

checkOneItems:ResourceItem{mName='layoutManager', mType=attr, mStatus=1}

checkOneItems:ResourceItem{mName='item_touch_helper_swipe_escape_velocity', mType=dimen, mStatus=1}

checkOneItems:ResourceItem{mName='item_touch_helper_max_drag_scroll_per_frame', mType=dimen, mStatus=1}

结合调用处dataSet.checkItems来看，这个逻辑应该是检查一个dataSet内是否有资源重名，一个dataSet一般有一个res目录，但也可以定义多个res目录，当定义多个res目录时，资源就有可能重复，这个检查逻辑可能主要处理这种情况，但我们添加了自己的代码把这个逻辑给禁用了，也就是允许有重复!

接着往下看

// loop on all the data items.

for (String dataItemKey : dataItemKeys) {

​    if (requiresMerge(dataItemKey)) {

​        // get all the available items, from the lower priority, to the higher

​        // priority

​        List<I> items = Lists.newArrayListWithExpectedSize(mDataSets.size());

​        for (S dataSet : mDataSets) {

​            // look for the resource key in the set

​            ListMultimap<String, I> itemMap = dataSet.getDataMap();

​            List<I> setItems = itemMap.get(dataItemKey);

​            items.addAll(setItems);

​        }

​        mergeItems(dataItemKey, items, consumer);

​        continue;

​    }

dataItemKeys就是将所有的ResourceSet中的item添加到一个set里面，有没有可能在添加时就去重了呢？dataItemKey表示的是资源的什么属性呢？查看日志

ResourceMerger:requiresMerge:dimen-godzillaui-nxhdpi-v4/list_preferred_item_height

ResourceMerger:requiresMerge:drawable/preference_item_bg

最后一列就是dataItemKey，从这来看dataItemKey就是type+qualifier/name，应该可以唯一标示一个资源项。

接着看  requiresMerge的实现

@Override

protected boolean requiresMerge(@NonNull String dataItemKey) {

​    new RuntimeException("ResourceMerger:requiresMerge:" + dataItemKey).printStackTrace();

​    return dataItemKey.startsWith("declare-styleable/");

}

declare-styleable类型的资源会走请求合并的逻辑，合并的逻辑是首先将所有DataSet中的相同的declare-styleable资源收集起来，然后调用mergeItems，这两段逻辑应该是把所有相同的declare-styleable合并起来，看看具体是怎么合并的。

@Override

protected void mergeItems(

​        @NonNull String dataItemKey,

​        @NonNull List<ResourceItem> items,

​        @NonNull MergeConsumer<ResourceItem> consumer) throws MergingException {

​    ...

​    try {

​        if (touched || (previouslyWrittenItem == null && !removed)) {

​            ...

​            // loop through all the items and gather a unique list of nodes.

​            // because we start with the lower priority items, this means that attr with

​            // format inside declare-styleable will be processed first, and added first

​            // while the redundant attr (with no format) will be ignored.

​            Set<String> attrs = Sets.newHashSet();

​            for (ResourceItem item : items) {

​                System.err.println("ResourceMerger:mergeItems:item:" + item);

​                if (!item.isRemoved()) {

​                    Node oldDeclareStyleable = item.getValue();

​                    if (oldDeclareStyleable != null) {

​                        NodeList children = oldDeclareStyleable.getChildNodes();

​                        for (int i = 0; i < children.getLength(); i++) {

​                            Node attrNode = children.item(i);

​                            System.err.println("ResourceMerger:mergeItems:attrNode:" + attrNode);

​                            ...

​                            String name = nameAttr.getNodeValue();

​                            if (attrs.contains(name)) {

​                                continue;

​                            }

​                            // duplicate the node.

​                            attrs.add(name);

​                            Node newAttrNode = NodeUtils.duplicateNode(document, attrNode);

​                            declareStyleableNode.appendChild(newAttrNode);

​                            System.err.println("ResourceMerger:mergeItems:appendChild:" + newAttrNode);

​                        }

​                    }

​                }

​            }

​            ...

​    } catch (ParserConfigurationException e) {

​        throw MergingException.wrapException(e).build();

​    }

}

合并的过程是将相同的declare-styleable合并成一个，定义的属性合并成一个集合，名称相同的属性，取第一个，后续的会被忽略，接着再看整个资源的合并过程，并且加入Log

// for each items, look in the data sets, starting from the end of the list.

I previouslyWritten = null;

I toWrite = null;

/*

* We are looking for what to write/delete: the last non deleted item, and the

* previously written one.

 */

boolean foundIgnoredItem = false;

setLoop: for (int i = mDataSets.size() - 1 ; i >= 0 ; i--) {

​    S dataSet = mDataSets.get(i);

​    System.err.println("loop:" + dataSet);

​    // look for the resource key in the set

​    ListMultimap<String, I> itemMap = dataSet.getDataMap();

​    List<I> items = itemMap.get(dataItemKey);

​    if (items.isEmpty()) {

​        continue;

​    }

​    System.err.println("getItems for:" + dataItemKey + ":" + items);

​    // The list can contain at max 2 items. One touched and one deleted.

​    // More than one deleted means there was more than one which isn't possible

​    // More than one touched means there is more than one and this isn't possible.

​    for (int ii = items.size() - 1 ; ii >= 0 ; ii--) {

​        I item = items.get(ii);

​        if (consumer.ignoreItemInMerge(item)) {

​            foundIgnoredItem = true;

​            continue;

​        }

​        if (item.isWritten()) {

​            assert previouslyWritten == null;

​            previouslyWritten = item;

​        }

​        if (toWrite == null && !item.isRemoved()) {

​            toWrite = item;

​        }

​        if (toWrite != null && previouslyWritten != null) {

​            break setLoop;

​        }

​    }

}

System.err.println("foundIgnoredItem:" + foundIgnoredItem + ",previouslyWritten:" + previouslyWritten + ",toWrite:" + toWrite);

// done searching, we should at least have something, unless we only

// found items that are not meant to be written (attr inside declare styleable)

assert foundIgnoredItem || previouslyWritten != null || toWrite != null;

if (toWrite != null && !filterAccept(toWrite)) {

​    toWrite = null;

}

//noinspection ConstantConditions

if (previouslyWritten == null && toWrite == null) {

​    continue;

}

// now need to handle, the type of each (single res file, multi res file), whether

// they are the same object or not, whether the previously written object was

// deleted.

if (toWrite == null) {

​    System.err.println("toWrite == null");

​    // nothing to write? delete only then.

​    assert previouslyWritten.isRemoved();

​    consumer.removeItem(previouslyWritten, null /*replacedBy*/);

} else if (previouslyWritten == null || previouslyWritten == toWrite) {

​    System.err.println("previouslyWritten == null || previouslyWritten == toWrite");

​    // easy one: new or updated res

​    consumer.addItem(toWrite);

} else {

​    System.err.println("else");

​    // replacement of a resource by another.

​    // force write the new value

​    toWrite.setTouched();

​    consumer.addItem(toWrite);

​    // and remove the old one

​    consumer.removeItem(previouslyWritten, toWrite);

}

整个合并逻辑比较长，一步一步看，先加Log，看看整个流程怎么走，再逐个点细看

loop:ResourceSet{debug, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/src/debug/res]}

loop:ResourceSet{main, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/src/main/res, /home/chenshun/AndroidStudioProjects/Testor/app/build/generated/res/rs/debug, /home/chenshun/AndroidStudioProjects/Testor/app/build/generated/res/resValues/debug]}

getItems for:drawable-nxhdpi-v4/action_button_search_normal_light:[ResourceItem{mName='action_button_search_normal_light', mType=drawable, mStatus=1, mLibraryName=null}]

consumer.ignoreItemInMerge(item)=false

item.isWritten()=false

toWrite == null && !item.isRemoved()=true

toWrite != null && previouslyWritten != null=false

loop:ResourceSet{0.1, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.miui.support/core/0.1/res]}

getItems for:drawable-nxhdpi-v4/action_button_search_normal_light:[ResourceItem{mName='action_button_search_normal_light', mType=drawable, mStatus=1, mLibraryName=com.miui.support:core:0.1}]

consumer.ignoreItemInMerge(item)=false

item.isWritten()=false

toWrite == null && !item.isRemoved()=false

toWrite != null && previouslyWritten != null=false

loop:ResourceSet{24.2.1, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/recyclerview-v7/24.2.1/res]}

loop:ResourceSet{24.2.1, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/support-core-ui/24.2.1/res]}

loop:ResourceSet{24.2.1, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/support-compat/24.2.1/res]}

loop:GeneratedResourceSet{debug$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/src/debug/res]}

loop:GeneratedResourceSet{main$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/src/main/res, /home/chenshun/AndroidStudioProjects/Testor/app/build/generated/res/rs/debug, /home/chenshun/AndroidStudioProjects/Testor/app/build/generated/res/resValues/debug]}

loop:GeneratedResourceSet{0.1$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.miui.support/core/0.1/res]}

loop:GeneratedResourceSet{24.2.1$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/recyclerview-v7/24.2.1/res]}

loop:GeneratedResourceSet{24.2.1$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/support-core-ui/24.2.1/res]}

loop:GeneratedResourceSet{24.2.1$Generated, sources=[/home/chenshun/AndroidStudioProjects/Testor/app/build/intermediates/exploded-aar/com.android.support/support-compat/24.2.1/res]}

foundIgnoredItem:false,previouslyWritten:null,toWrite:ResourceItem{mName='action_button_search_normal_light', mType=drawable, mStatus=1, mLibraryName=null}

ResourceMerger:filterAccept:ResourceItem{mName='action_button_search_normal_light', mType=drawable, mStatus=1, mLibraryName=null}

previouslyWritten == null || previouslyWritten == toWrite

从Log中可以看到，在两个资源目录中找到了相同的资源，取了第一个，后一个就被忽略掉了。基本上可以确定，在这个地方打警告信息就可以了