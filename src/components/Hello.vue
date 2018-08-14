<template>
  <div class="fluid container">
    <div class="form-group form-group-lg panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title">Sortbale control</h3>
      </div>
      <div class="panel-body">
        <div class = "checkbox">
          <label><input type = "checkbox" v-model="editable">Enable drag and drop</label>      
        </div>
        <button type="button" class="btn btn-default" @click="orderList">Sort by original order</button>
      </div>
    </div>

    <div class="col-md-3" v-for="list in lists" :key="list.id" >
      <h2>{{ list.name }}</h2>
      <draggable element="span" :list="list.tasks" :options="dragOptions" @start="isDragging=true" @end="changeStatuses" > 
        <transition-group name="no" class="list-group" tag="ul">
          <li class="list-group-item" v-for="(element, index) in list.tasks" :key="element.id"> 
            {{element.name}}
            <i :class="'fa fa-check'" @click="check(index, list.tasks)" aria-hidden="true"></i>
            <i :class="element.status === 'active' ? 'fa fa-pause' : 'fa fa-play'" @click="fixed(element, index, list.tasks)" aria-hidden="true"></i>
            <span class="badge">{{element.time | timeConvert}}</span>
          </li> 
        </transition-group>
      </draggable>
    </div>

    <div class="list-group col-md-3">
      <pre>{{listString}}</pre>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
var people = [
  {
    id: 0,
    name: 'Sherzod',
    tasks: [
      {
        name: 'Js',
        id: 1,
        userId: 1,
        time: 0,
        status: 'active',
        fixed: false
      }
    ]
  },
  {
    id: 2,
    name: 'Jasur',
    tasks: [
      {
        name: 'Php',
        id: 2,
        userId: 2,
        time: 0,
        status: 'active',
        fixed: false
      }
    ]
  },
  {
    id: 3,
    name: 'Nurbek',
    tasks: [
      {
        name: 'Html',
        id: 3,
        userId: 3,
        time: 0,
        status: 'active',
        fixed: false
      }
    ]
  }
];

export default {

  name: 'hello',
  components: {
    draggable,
  },

  data () {

    return {
      // lists: message.map( (item) => { return item.map( (name,index) => {return {name, order: index+1, fixed: false}; })}),
      lists: people,
      editable:true,
      isDragging: false,
      delayedDragging:false
    };

  },

  created: function(lists) {
    
    setInterval(function() {

      for (var list of people) {

        for (var tasks of list.tasks) {

          if (tasks.status === 'active') {

            tasks.time++;

          }  
          
        }

      }

    }, 1000);

  },

  methods:{

    orderList () {

      this.list = this.list.sort((one,two) =>{return one.order-two.order; });

    },

    onMove ({relatedContext, draggedContext}) {

      const relatedElement = relatedContext.element;
      const draggedElement = draggedContext.element;
      
      this.changeStatuses();

      return true;

    },

    timeUpdate (lists) {
      
      setInterval(function() {

        for (var list of lists) {

          for (var tasks of list.tasks) {
            
            tasks.time++;
            
          }
        }
      }, 1000);

    },

    changeStatuses () {

      this.isDragging = false;

      people  = [].map.call(people,(person) => {

      var fixed = true;

      person.tasks = [].map.call(person.tasks, (task, index) => {

          task.status = 'disabled';

          if (index === 0 && !task.fixed) {

            task.status = 'active';
            fixed = false;

          } else if (!task.fixed && fixed) {

            task.status = 'active';
            fixed = false;

          }

          return task;

        });

        return person;

      });
    },

    fixed: function(element, index, tasks) {
      
      var task;
      
      if (element.status === 'disabled' && index > 0) {

        element.fixed = false;
        task = element;
        [].splice.call(tasks, index, 1);
        [].unshift.call(tasks, task);

      } else {

        element.fixed = !element.fixed;

      }
      this.changeStatuses();

    },
    
    check: function(index, tasks) {

      if (confirm()) {

        [].splice.call(tasks, index, 1);
        this.changeStatuses();

      }
      
    }

   
  },

  computed: {

    dragOptions () {

      return  {
        animation: 0,
        group: 'description',
        disabled: !this.editable,
        ghostClass: 'ghost'
      };

    },

    listString() {

      return JSON.stringify(this.lists, null, 2);

    }
  },

  watch: {

    isDragging (newValue) {
      if (newValue) {

        this.delayedDragging = true;
        return;

      }
      this.$nextTick( () => {

           this.delayedDragging = false;

      });
    }

  },
  
  filters: {
    timeConvert: function (d) {
          d = Number(d);
        var day = Math.floor(d / (3600 * 60));
        var h = Math.floor(d / 3600);
        var m = Math.floor(d % 3600 / 60);
        var s = Math.floor(d % 3600 % 60);

        var dayDisplay = day > 0 ? day + " d, " : "";
        var hDisplay = h > 0 ? h +  " h, " : "";
        var mDisplay = m > 0 ? m +  " m, " : "";
        var sDisplay = s > 0 ? s +  " s" : "";
        return dayDisplay + hDisplay + mDisplay + sDisplay; 
    }
}

}
</script>

<style scoped>

.flip-list-move {
  transition: transform 0.5s;
}

.no-move {
  transition: transform 0s;
}

.ghost {
  opacity: .5;
  background: #C8EBFB;
}

.list-group {
  min-height: 20px;
}

.list-group-item {
  cursor: move;
}

.list-group-item i{
  cursor: pointer;
}

i.fa {
  margin-left: 5px;
  min-width: 14px;
  font-size: 16px;
  float: right;
}

.fa-check {
  color: #06f812;
}

</style>
