# easy-backfilling
Python script for PEC1 for HPC

        from tabulate import tabulate
        
        def easy_backfilling(jobs, total_cpus):
            jobs.sort(key=lambda x: x['arrival'])
            current_time = 0
            cpus = {i: [] for i in range(1, total_cpus + 1)}
            resource_usage = 0
            total_time = 0
            jobs = jobs.copy()
        
            while jobs:
                first_job = None
                for job in jobs:
                    if job['arrival'] <= current_time:
                        first_job = job
                        break
                if not first_job:
                    current_time = jobs[0]['arrival'] if jobs else current_time
                    continue
        
                available_cpus = []
                for cpu in cpus:
                    if not cpus[cpu] or cpus[cpu][-1][1] <= current_time:
                        available_cpus.append(cpu)
                if len(available_cpus) >= first_job['cpus']:
                    start = current_time
                    end = start + first_job['runtime']
                    assigned_cpus = available_cpus[:first_job['cpus']]
                    for cpu in assigned_cpus:
                        cpus[cpu].append((start, end, first_job['id']))
                    resource_usage += first_job['cpus'] * first_job['runtime']
                    total_time = max(total_time, end)
                    jobs.remove(first_job)
                    continue
                else:
                    backfilled = False
                    for job in jobs:
                        if job == first_job or job['arrival'] > current_time:
                            continue
                        job_end = current_time + job['runtime']
                        if job_end > (current_time + first_job['runtime']):
                            continue
                        available_for_job = []
                        for cpu in cpus:
                            if not cpus[cpu] or cpus[cpu][-1][1] <= current_time:
                                available_for_job.append(cpu)
                        if len(available_for_job) >= job['cpus']:
                            start = current_time
                            end = start + job['runtime']
                            assigned_cpus = available_for_job[:job['cpus']]
                            for cpu in assigned_cpus:
                                cpus[cpu].append((start, end, job['id']))
                            resource_usage += job['cpus'] * job['runtime']
                            total_time = max(total_time, end)
                            jobs.remove(job)
                            backfilled = True
                            break
                    if not backfilled:
                        current_time += 1
        
            schedule = {}
            for cpu in cpus:
                schedule[cpu] = sorted(cpus[cpu], key=lambda x: x[0])
                schedule[cpu] = [(job_id, start, end) for (start, end, job_id) in schedule[cpu]]
        
            return schedule, resource_usage, total_time
        
        def print_schedule_table(schedule, total_cpus, max_time=15):
            grid = []
            for cpu in range(1, total_cpus + 1):
                cpu_jobs = schedule.get(cpu, [])
                time_slots = [' '] * max_time
                for job in cpu_jobs:
                    job_id, start, end = job
                    for t in range(start, end):
                        if t <= max_time:
                            time_slots[t - 1] = str(job_id)
                grid.append([f"CPU {cpu}"] + time_slots)
        
            headers = ["Time\\CPU#"] + [str(i) for i in range(1, max_time + 1)]
            print(tabulate(grid, headers=headers, tablefmt="grid"))
        
        jobs = [
            {'id': 1, 'arrival': 1, 'runtime': 2, 'cpus': 4},
            {'id': 2, 'arrival': 2, 'runtime': 3, 'cpus': 2},
            {'id': 3, 'arrival': 3, 'runtime': 5, 'cpus': 1},
            {'id': 4, 'arrival': 3, 'runtime': 2, 'cpus': 2},
            {'id': 5, 'arrival': 4, 'runtime': 1, 'cpus': 5},
            {'id': 6, 'arrival': 7, 'runtime': 4, 'cpus': 5},
            {'id': 7, 'arrival': 9, 'runtime': 4, 'cpus': 3},
            {'id': 8, 'arrival': 9, 'runtime': 2, 'cpus': 1},
            {'id': 9, 'arrival': 11, 'runtime': 3, 'cpus': 3},
            {'id': 10, 'arrival': 12, 'runtime': 1, 'cpus': 2},
        ]
        
        total_cpus = 6
        max_time = 15  # Fixed to match the exercise's 15-time-slot table
        schedule, resource_usage, dynamic_time = easy_backfilling(jobs, total_cpus)
        
        print_schedule_table(schedule, total_cpus)
        
        # Calculate utilization based on fixed max_time=15
        total_node_hours_available = total_cpus * max_time
        utilization = resource_usage / total_node_hours_available
        
        print(f"\nResource Utilization Formula:")
        print(f"Utilization = Total Node-Hours Used / (Total CPUs × Total Time)")
        print(f"             = {resource_usage} / ({total_cpus} × {max_time})")
        print(f"             = {resource_usage} / {total_node_hours_available}")
        print(f"             = {utilization * 100:.2f}%")
