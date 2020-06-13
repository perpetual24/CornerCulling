# CornerCulling
Fast and maximally accurate culling method. Proof of concept in C++ and UE4.
Wallhack Penicillin.

Runtime demonstration:
https://youtu.be/SHUXDR0hleU

Accuracy demonstration:
https://youtu.be/tzrIXcdYQJE

Also accounts for client latency, allowing the server to maximally cull according to where a laggy player could be in the future.

## Pitch

While not as rage-inducing as a blatant aim-botter, a wallhacker inflicts great mental strain on their victims, making them constantly uneasy and suspicious. Many serious FPS players, including myself, have spent hours staring at replays, trying to catch them red-handed. But at some point, it all becomes too exhausting.

Furthermore, wallhacks are often too subtle to detect with simple heuristics, making detection and punishment strategies costly and ineffective.

So, instead of detecting wallhacks, many games have adopted a prevention strategy. With this strategy, the game server independently calculates if a player can see an enemy. If the player cannot see the enemy, then the server does not send the enemy location to the player. Thus, even if a malicious player tampers with their game or memory, they will not be able to find the locations of enemies behind walls.

Unfortunately, modern implementations of this idea are inaccurate or slow. I have developed a solution that is both perfectly accurate and faster than currently deployed solutions.


## Major Task:  
Big refactor ¯\\\_(ツ)_/¯  
Move occlusion logic to OcclusionController class.  
Disentangle VisibilityPrisms from players and occluding objects.  

Refactor design doc:  
    Culling controller handles all culling. New culling loop.  
    
```python
# Get line segments needed to calculate LOS between each player and their enemies.
for player_i in player_indicies:  
    for enemy_i in player_indicies:
        if (get_team(player_i) != get_team(enemy_i) and
            (almost_visible(player_i, enemy_i) or
             ((cull_this_tick() and
              potentially_visible(player_i, enemy_i)
             )
            )
           ):  
            LOS_segments_queue.push(get_LOS_segments(player_i, enemy_i))

# Try to block segments with occluding planes in each segments' player's cache.
# This case should be common and fast.
for segments in LOS_segments_queue:
    for plane in occluding_plane_caches[segments.player_i]:  
        if (intersects(segments.left, plane) or
            intersects(segments.right, plane)
           ):
            blocked_queue.push(segment)
            break
    LOS_segment_queue_2.push(segment)

# Get occluding planes for all potentialy visible occluders.
for player_i in player_indicies:  
    for occluder in occluding_prisms:  
        if potentially_visible(player_i, occluder):  
            occluding_plane_queue.push(get_occluding_finite_plane(player_i, occluder))

# Check remaining LOS segments against all occluding planes in the queue.
for segments in LOS_segments_queue_2:
    for plane in occluding_plane_queue:  
        if (intersects(segment.left, plane) or
            intersects(segment.right, plane)
           ):
            blocked_queue.push(segment)
            update_cache_LRU(occluding_plane_caches[segments.player_i], plane)
            break

for segments in blocked_queue:
    hide(segments.player_i, segments.enemy_i)
reset_queues()
```
               
## Other Tasks (in no order):
1)  Implement (or hack together) potentially visible sets to cull enemies and occluding objects.
2)  Test performance of occluding surfaces, aka 2D walls instead of boxes
3)  Calculate Z visibility by projecting from top of player to top of wall in the direction
    of the enemy. If this angle hits below the top of the enemy, or hits the semicircle bounding the top
    of the enemy, reveal the enemy.
4)  Reach out to more FPS game developers.
5)  Continue researching graphics community state of the art.
6)  What to do about a wallhacking Jet with a lag switch? Cull based on trust factor!
8)  Make enemy lingering visibility adaptive only when server is under load.
9)  Use existence-based predication for caches. Test LRU, k-th chance, and random replacement algorithms.
10) Change "RelevantCorners" to just "LeftAndRightCorners" for readability.
11) Design doc opimizations for large Battle Royale type games.
    No culling until enough players die. PVS filter players and occluders.

## Miscellanea
Unbeknownst to me at first (but unsurprisingly), my idea is basically just shadow culling,
which graphics researchers documented in 1997. <br />
https://www.gamasutra.com/view/feature/3394/occlusion_culling_algorithms.php?print=1 <br />
I suspect that I could incorporate improvements made in the past 20 years.
