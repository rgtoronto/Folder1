### Application Properties:
    Missing "spark.executor.cores": "8" from GUI
    
NUM_STEPS=$1
STEP_OFFSET=$2
STEP_ITERATIONS=$3

## To check space usage distribution on machine:
    du -sh *
    
## To check space usage on each machine
    ./check_space.sh