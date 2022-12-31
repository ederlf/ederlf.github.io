+++
title = "Parsing the RTA_MULTIPATH attribute from Rtnetlink"
date = 2018-01-25
math = false
highlight = false
categories = ["Programming"]
tags = ["C", "Rtnetlink", "RTA_MULTIPATH"]

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = ""

+++

I am developing an application that relies on the detection of changes to the Linux kernel route tables. For the job, one can use [Rtnetlink](http://man7.org/linux/man-pages/man7/rtnetlink.7.html) for reading and modification of the routing tables. For example, the *RTM_NEWROUTE* message can be used to add a new route or read from the netlink socket, indicating a new route was added to the route table. 

Netlink messages are usually composed of attributes which contain the information associated to the message type. For my use case, I need to obtain nexthop of new added routes. There are two attributes where next hops can be found: 


* The usual attribute for the task is *RTA_GATEWAY* in a single route message. it is not hard to find [examples](https://gist.github.com/cl4u2/5204374#file-route_dump-c-L102) for that case. The next hop data in that case is fairly simply too and points directly the value of the IP address of the next hop. 


* The second case where next hop information is present is for the *RTA_MULTIPATH* attribute. Unfortunately, the information is not that straight forward and the available examples are not very simple. This post aims to be a simpler reference for anyone looking for the right way to parse *RTA_MULTIPATH*. 


The example will consider [this loop](https://gist.github.com/cl4u2/5204374#file-route_monitor-c-L59). The idea is too add the case that handles *RTA_MULTIPATH*. 

The data obtained from *RTA_MULTIPATH* is an array of [struct rtnexthop](https://elixir.bootlin.com/linux/v4.11/source/include/uapi/linux/rtnetlink.h#L339). While this struct describes all necessary information about next hops, it is not necessary to retrieve the data. For the task, rtnetlink provides macros to manipulate next hop information in the same fashion as messages.


The code below shows how the next hops can be retrieved. 


```C
if (route_attribute->rta_type == RTA_MULTIPATH){
    /* Get RTA_MULTIPATH data */
    struct rtnexthop *nhptr = (struct rtnexthop*) RTA_DATA(route_attribute);
    int rtnhp_len = RTA_PAYLOAD(route_attribute);
    /* Is the message complete? */
    if (rtnhp_len < (int) sizeof(*nhptr) || 
        nhptr->rtnh_len > rtnhp_len){
        continue;
    }
    /* Get the size of the attributes */
    attrlen = rtnhp_len - sizeof(struct rtnexthop);
    if (attrlen) {
        /* Retrieve attributes */
        struct rtattr *attr = RTNH_DATA(nhptr);
        for (; RTA_OK(attr, attrlen); attr = RTA_NEXT(attr, attrlen)) {
            if (attr->rta_type == RTA_GATEWAY) {
                /* Next hops from RTA_MULTIPATH are 
                 * contained in RTA_GATEWAY attributes! 
                 */
                uint32_t nh = *(uint32_t*) RTA_DATA(attr);
                fprintf(stdout, "%x", nh);  
        }
    }
}

```

The initial step is to get the *RTA_MULTIPATH*, similarly to other attributes using the RTA_DATA macro. A check for the correct size of the message follows, in case the message was not fully read from the socket. If the payload of the attribute is larger than its header, the data is retrieved using RTNH_DATA. What comes next is the most interesting part. Inside the next hop data are...guess what? More attributes! The next hop is then found in the same fashion as the [gist](https://gist.github.com/cl4u2/5204374#file-route_monitor-c-L95).

I hope this small example may help anyone looking for a way to retrieve information from the *RTA_MULTIPATH* attribute.  
