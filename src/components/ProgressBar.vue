<template>
    <div class="pl_container">
        <div :style="fillerStyles">
            <span class="label">{{ `${completed}%` }}</span>
        </div>
        <div class="mpgs">
        </div>
    </div>
</template>
  
<script>
import { computed } from "vue";
export default {
    name: "ProgressBar",
    props: {
        bgcolor: String,
        completed: String,
    },
    setup(props) {
        const fillerStyles = computed(() => {
            return {
                height: "100%",
                width: `${props.completed}%`,
                backgroundColor: props.bgcolor,
                transition: "width 1s ease-in-out",
                borderRadius: "10px",
                textAlign: "right",
                "white-space": "nowrap",

            };
        });

        return { fillerStyles };
    },
};
</script>
  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pl_container {
    height: 20px;
    width: 50%;
    background-color: #e0e0de;
    border-radius: 50px;
    margin: 10px;
    margin-right: 10px;
}

.label {
    padding: 5px;
    color: white;
    font-weight: bold;
}

.mpgs {
    height: "100%";
    width: 206%;
    background-Color: red;
    transition: "width 1s ease-in-out";
    border-Radius: 10px;
    text-Align: "right";
    white-space: "nowrap";
}

/* Estilos para pantallas pequeñas */
@media (max-width: 768px) {
    .container {
        height: 20px;
        width: 70%;
        background-color: #e0e0de;
        border-radius: 50px;
        margin: 50px;
        margin-right: 5px;
    }
}
</style>
  