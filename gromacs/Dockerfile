apiVersion: kubeflow.org/v2beta1
kind: MPIJob
metadata:
  name: gromacs-test
  labels:
    kueue.x-k8s.io/queue-name: user-queue
spec:
  slotsPerWorker: 1
  runPolicy:
    cleanPodPolicy: Running
  sshAuthMountPath: /home/mpiuser/.ssh
  mpiReplicaSpecs:
    Launcher:
      replicas: 1
      template:
        spec:
          containers:
          - image: intamixx/gromacs-2023.1
            name: mpi-launcher
            securityContext:
              runAsUser: 1000
            command:
            - mpirun
            args:
            - -np
            - "2"
            - gmx_mpi
            - mdrun
            - -ntomp
            - "1"
            - -s
            - /scratch/benchRIB.tpr
            - -g
            - /scratch/md.log
            - -e
            - /scratch/ener.edr
            - -cpo
            - /scratch/state.cpt
            - -c
            - /scratch/confout.gro
            - -resethway
            resources:
              limits:
                cpu: 1
                memory: 1Gi
    Worker:
      replicas: 2
      template:
        spec:
          containers:
          - image: intamixx/gromacs-2023.1
            name: mpi-worker
            resources:
              limits:
                cpu: 1
                memory: 4Gi
