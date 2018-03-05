

how to recover from sidechain
v0.1.14 enforced the total difficulty check. if the block found by node , the total diff is less than the main net, the new block propagation will be rejected.

steps to identify if node are in sidechain.

    turn on sync debug in config.xml
    image

    check if log have "diff-fail" msg.
    image

    revert your db to main net.
    ./aion.sh -r "your_block_number_when_forked"

    restart kernel to check if block number back to normal.
