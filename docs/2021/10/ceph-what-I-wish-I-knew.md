# Ceph what I wish I knew

Below are some points of things that I wish I knew before jumping into Ceph. (I will update this as we go further along this journey.)

* Don't fall into the trap of using consumer drives. I think a lot of people (myself included) think of Ceph as a way to use consumer hardware to achieve high performance and highly fault tolerant storage. We ended up falling into this trap starting with Samsung Evo SSD's. Testing against Intel Enterprise drives with super capacitor backed cache gave a 20-30% improvement.
* If using SSD change the disk scheduler to noop/none. Doing this gave a 10-20% performance bump and also dropped latency for requests.