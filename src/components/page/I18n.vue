<!-- 国际化组件 -->
<template>
    <section>
        <div class="crumbs">
            <!-- 面包屑 -->
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <!-- $t是方法调用 -->
                    <i class="el-icon-orange">{{ $t('i18n.breadcrumb') }}</i>
                    <!-- $t用于获取指定键名的翻译文本 -->
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <span>{{ $t('i18n.tips') }}</span>
            <!-- 通过this.$i18n.locate来进行语言的切换 -->
            <el-button type="primary" @click="$i18n.locale = $i18n.locale === 'zh' ? 'en' : 'zh';">
                {{ $t('i18n.btn') }}
            </el-button>
            <div class="list">
                <h2>{{ $t('i18n.title1') }}</h2>
                <p>{{ $t('i18n.p1') }}</p>
                <p>{{ $t('i18n.p2') }}</p>
                <p>{{ $t('i18n.p3') }}</p>
            </div>
            <h2>{{ $t('i18n.title2') }}</h2>
            <div>
                <!-- 使用组件的方式处理 path指定标签的内容 tag 生产标签 -->
                <i18n path="i18n.info" tag="p">
                    <!-- 通过 place 来指定出现在 HTML 中的位置 -->
                    <a place="action" href="https://element.eleme.cn/2.0/#/zh-CN/component/i18n">{{ $t('i18n.value') }}</a>
                </i18n>
            </div>
        </div>
    </section>
</template>


<style scoped>
.list {
    padding: 30px 0;
}

.list p {
    margin-bottom: 20px;
}

a {
    color: #409eff;
}
</style>
