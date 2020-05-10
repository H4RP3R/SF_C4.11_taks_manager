<template>
<div>
    <b-alert variant="success"
        dismissible
        @dismissed="$emit('disable')"
        show>{{ message }}</b-alert>
</div>
</template>

<script>
export default {
    props: ['message'],
};
</script>
