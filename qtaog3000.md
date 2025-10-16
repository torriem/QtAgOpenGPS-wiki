# QtAOG 3000

This document is describes a possible future implemenation of QtAgOpenGPS

## Features
 * Any vehicle combination, including fixed and arbitrary trailed implements.
   * Vehicles is primarily steered
   * Implements are containers for toolbars, or may just be a means of attaching to other implements. For example an anhydrous ammonia tank trailer, or an air drill tank
   * Toolbars attached to implements, contain sections
   * Can have multiple implements, arbitrarily
 * Multiple GNSS inputs.  The unit attached to the tractor is primary for steering.  Implement GNSS for more precise mapping, recording, and also implement steering
 * Multiple toolbars
   * Each toolbar is fixed to an axle
   * Each toolbar is a container for sections (which could be planter row units)
   * Toolbar holds default settings, including which layer to use for coverage and recording, and default locations.
   * Each section can be arbitrarily placed in space relative to toolbar
 * Jobs
   * Fields, tracks, boundaries, etc only need be created once
   * Each field operation is its own job that shares the boundaries and tracks
   * Which variable rate map to use
   * Variable rate layer to section assignments
 * Coverage Layers
   * Multiple layers
   * Assigned per toolbar, or even per section
   * Implements can write data to the coverage layer, recorded per triangle.
     * Yield, fuel, achieved rate, etc
     * Not sure how to deal with delayed writes, as you'd need for yield recording
 * Variable Rate Maps
   * Multiple layers, assigned to toolbars or individual sections by the job I suppose
   * Ultimately provides a rate for a particular section
 * Autosteer
   * Contour (based on a coverage layer, would have to be particular to a job?)
   * Curves with the following types
     * A + heading
     * AB Lines
     * Curves
       * From boundaries, etc
     * Circle
       * Center point, concentric out from there
       * Center point, outer-most radius, work in from there
   * Nudge, remark?
 * AgIO
   * Direct section control
   * GPS in and out
   * IMU stuff
 * Plugins
   * Rate control
   * ISOBUS Integration
     * Current coverage and variable rate map as a Task Controller
     * Create QtAOG implement from ISOBUS implement information, including toolbar and sections using ISOBUS-provided numbers for hitch length, section numbers, widths, etc
   * autosteer algorithms
   * youturn algorithms?
   * track types?

## Software Architecture
 * Core abstract components
   * Vehicles and Implements
     * axle - a point around which a body moves linearly or pivots
     * steering axle - abstraction of the steering wheels position and angle
     * articulation joint
     * hitch - a fixed point, relative to the axle which other things can connect, pivoting or rigid.  IE, a drawbar or a 3pt hitch
     * vehicle - a collection of hitches, an axle (or two with articulation), steering axle, with GPS and steering angle as inputs. Only one vehicle permitted at a time. Only a steering axle or an articulation joint allowed (mapped to the steering angle sensor).
     * tongue - a fixed point relative to an axle that connects to other hitches
     * toolbar - a linear line, relative to an axle to which sections are attached
     * section - attached to a toolbar, with individual offsets and width
     * implement - an axle, a tongue, a collection of hitches, and a collection of toolbars and sections and optional implement GNSS
   * Fields
   * Autosteer
     * Abstract line segment-based steering, using pure pursuit, stanley, or some other algorithm
     * Different track types devolve into AB line segments, or in other words, curves
     * Path planning, any way to make it common between types?  All 
     * Track types
## Program flow using GNSS fix
 * fixes arrive at a vehicle and then propagate through the hitches and tongues to the implements where all the positions of axles, toolbars, and sections are.
 * if an implement has a GNSS input, that is used to calculate all positions of that implement and then they flow through hitches to other connected implements.
 * coverage, section control, and rate control is then calculated based on those positions
## autosteer
 * vehicle
 * implement steering

