<template>
  <div class="mod-home">
    <h3>Project Introduction</h3>
    <ul>
      <li>Developed a distributed e-commerce system based on Spring Boot and Spring Cloud, used Docker to manage multiple middleware and Nginx for dynamic and static separation, reverse proxy, and load balancing.</li>
      <li>Introduced a complete set of microservice governance solution: Nacos as a registration and configuration center, Gateway as a gateway, Feign for remote call, Ribbon for load balancing, Sentinel for flow protection, Sleuth and Zipkin as a tracing system.</li>
      <li>Solved most of the problems faced by a highly concurrent distributed system: Spring Cache and Redis as distributed cache, Elasticsearch for faster product retrieval time, Spring Session for session data sharing, thread pool and asynchronous task for stability and performance.</li>
      <li>Implemented generating/canceling order and locking/unlocking inventory using RabbitMQ delayed queue based on BASE theory and Flexible Transaction - message reliability and eventual consistency of distributed transactions.</li>
      <li>Completed product flash sale using Redisson distributed lock - Semaphore and MQ, which can process 50,000 QPS in one Tomcat server.</li>
    </ul>
    <h3>Links</h3>
    <ul>
      <li>Frontend Source Code: <a href="https://github.com/zli78122/gulimall_renren-fast-vue" target="_blank">https://github.com/zli78122/gulimall_renren-fast-vue</a></li>
      <li>Backend Source Code: <a href="https://github.com/zli78122/gulimall" target="_blank">https://github.com/zli78122/gulimall</a></li>
      <li>Github: <a href="https://github.com/zli78122" target="_blank">https://github.com/zli78122</a></li>
      <li>LinkedIn: <a href="https://www.linkedin.com/in/zhengyu-li-83b85119b/" target="_blank">https://www.linkedin.com/in/zhengyu-li-83b85119b/</a></li>
    </ul>
    <h3>Explanations</h3>
    <h4 style="margin-bottom: 10px">1. Relationships between SPU, SKU, Brand, Category, and Product Attribute</h4>
    <p class="homep">Understanding the following four concepts is crucial for you to understand the relationship between these entities.</p>
    <span class="homespan">1. Sales Attribute: The different attribute values of the sales attributes of a product will lead to differences in the sales price of this product.</span>
    <br />
    <span class="homespan" style="margin-left: 50px;">e.g. iPhone 12 Pro Max 256GB and iPhone 12 Pro Max 512GB have a different sales price. <span style="margin-left: 0; color: red;">So "capacity" is a sales attribute</span></span>
    <br />
    <span class="homespan">2. Regular Attribute: It will not affect the price of the product, like the length, weight, ppi of iPhone 12 Pro Max.</span>
    <br />
    <span class="homespan">3. SPU (Standard Product Unit): It is a type of goods, such as a type of iPhone, like iPhone 12 Pro Max.</span>
    <br />
    <span class="homespan">4. SKU (Stock keeping Unit): It is a specific product unit based on the SPU, like <span class="red">iPhone 12 Pro Max Gold+256GB</span>, <span class="red">iPhone 12 Pro Max Gold+512GB</span>, <span class="red">iPhone 12 Pro Max Blue+256GB</span>, <span class="red">iPhone 12 Pro Max Sliver+512GB</span>. 
      i.e. SKU is a combination of various Sales Attributes (color + capacity) of the SPU.</span>
    <br />
    <img src="~@/assets/img/ER.png" class="er">

    <h4 style="margin-top: 100px; margin-bottom: 10px">2. Flow Chart of Generating/Canceling Order and Locking/Unlocking Inventory (Delayed Queue, BASE Theory, Distributed Flexible Transaction)</h4>
    <span class="homespan"><strong>Locking Inventory</strong>: After the order is generated and before the user pays, the product purchased by current user will be locked in the inventory, which means that during this period, the product only belongs to the current user, and any other users can not purchase this product.</span>
    <br />
    <span class="homespan"><strong>Unlocking Inventory</strong>: When the order is paid or cancelled, the system will unlock the inventory.</span>
    <br />
    <img src="~@/assets/img/Order.png" class="order">
    
    <h4 style="margin-top: 100px">3. Flow Chart of Product Flash Sale (Distributed Lock - Semaphore, MQ)</h4>
    <span class="homespan"><strong>Flash Sale</strong>: Online store offers substantial discounts or promotions for a very short period of time. The main goals of a flash sale strategy are to get online shoppers to impulse buy, to increase short-term sales, or to sell your surplus stock.
    <span style="color: red;">So a flash sale means a very high amount of concurrency, which may reach millions of concurrency.</span></span>
    <br />
    <img src="~@/assets/img/FlashSale.png" class="flashsale">
  </div>
</template>

<script>
  export default {
  }
</script>

<style>
  .mod-home {
    line-height: 1.5;
  }
  .er {
    width: 1050px;
    height: 600px
  }
  .homep {
    font-size: 15px;
    margin-top: 0;
    margin-bottom: 5px;
  }
  .homespan {
    margin-left: 30px;
    font-size: 15px ;
  }
  span .red {
    margin-left: 10px;
    color: red;
  }
  .order {
    width: 1050px;
    height: 750px
  }
  .flashsale {
    width: 100%;
    margin-bottom: 50px;
  }
</style>