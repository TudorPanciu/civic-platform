# CIVIC Platform - Secure E-Voting & Civic Engagement

A comprehensive democratic participation portfolio platform featuring secure voting with automated reporting, social-style civic engagement, real-time communication, and production-ready administrative infrastructure. Built with production-grade security, performance optimization, and user experience design.

**Live Demo**: [https://civic-platform.org](https://civic-platform.org)  
**Architecture Documentation**: [Link to your public repository]

### Key Capabilities

🗳️ **Secure Voting System** - Anonymous ballot processing with tamper detection

📊 **Automated Reporting** - Interactive visualization with demographic analysis

🚀 **Dynamic Initiative Platform** - Social media-quality civic engagement 

🔒 **Multi-Layer Security** - Rate limiting, CSRF protection, audit logging

💬 **Real-Time Communication** - Encrypted chat with file sharing  

📧 **Automated Email Delivery Hub** -  Official branded communications with queue processing  

### Project Scale

- **~16,000 lines of code** across Python, JavaScript, HTML, and CSS
- **30+ interconnected database tables** with complex relationships
- **Comprehensive security** with OWASP-compliant protections
- **Real-time features** using WebSockets and dynamic interfaces
- **Platform email system** with queue processing and professional branding

## 🗳️ Secure Voting System

**Challenges:**

  1. Creating an admin-regulated ballot generation system, capitalizing on non-repudiation.
  2. Uplifting participation by providing enhanced accessibility and interactivity on all devices,
backed by reminders in the form of notifications.
  3. Developing tamper-resistant voting with auditing capabilities.
  4. Maintaining voter privacy.

**Technical Solutions:**

```python
class Ballot(Base):
    __tablename__ = 'ballots'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    type: Mapped[str] = mapped_column(String, nullable=False)
    title: Mapped[str] = mapped_column(String, nullable=False)
    threshold: Mapped[float] = mapped_column(Float, nullable=False)
    start: Mapped[datetime] = mapped_column(DateTime, nullable=False)
    end: Mapped[datetime] = mapped_column(DateTime, nullable=False)
    status: Mapped[str] = mapped_column(String, nullable=False, default="upcoming")
       

    items: Mapped[List["BallotItem"]] = relationship(back_populates='ballot', order_by="BallotItem.position")
    election_participations: Mapped[List["ElectionParticipation"]] = relationship(back_populates='ballot')
    question_participations: Mapped[List["QuestionParticipation"]] = relationship(back_populates='ballot')
    referendum_votes: Mapped[List["ReferendumVote"]] = relationship(back_populates='ballot')
    election_votes: Mapped[List["ElectionVote"]] = relationship(back_populates='ballot')
    election_result: Mapped[Optional["ElectionResult"]] = relationship(back_populates='ballot')
    referendum_result: Mapped[Optional["ReferendumResult"]] = relationship(back_populates='ballot')
    notifications: Mapped[List["BallotNotification"]] = relationship(back_populates='ballot')
    deleted: Mapped[Optional["DeletedBallot"]] = relationship(back_populates='ballot')
    electorate: Mapped[Optional["BallotElectorate"]] = relationship(back_populates='ballot')
    election_pyramid_graph: Mapped[Optional["ElectionPyramidGraph"]] = relationship(back_populates='ballot')

class BallotItem(Base):
    __tablename__ = 'ballot_items'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    ballot_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballots.id'), nullable=False)
    position: Mapped[int] = mapped_column(Integer, nullable=False)
    text: Mapped[str] = mapped_column(String, nullable=False)
    
  
    ballot: Mapped["Ballot"] = relationship(back_populates='items')
    referendum_votes: Mapped[List["ReferendumVote"]] = relationship(back_populates='item')
    election_votes: Mapped[List["ElectionVote"]] = relationship(back_populates='item')
    candidate_result: Mapped[Optional["CandidateResult"]] = relationship(back_populates='ballot_item')
    question_result: Mapped[Optional["QuestionResult"]] = relationship(back_populates='ballot_item')
    question_pyramid_graph: Mapped[Optional["QuestionPyramidGraph"]] = relationship(back_populates='ballot_item')
    question_participations: Mapped[Optional["QuestionParticipation"]] = relationship(back_populates='ballot_item')

class DeletedBallot(Base):
    # Admin accountability ensures one can't delete without taking responsibility
    __tablename__ = 'deleted_ballots'
    ballot_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballots.id'), nullable=False)
    user_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), nullable=False)
    reason: Mapped[str] = mapped_column(String, nullable=False)  # Required justification
    date: Mapped[datetime] = mapped_column(DateTime, default=datetime.now)

class BallotNotification(Base):
    # Participation reminders feature
    __tablename__ = 'ballot_notifications'
    user_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), nullable=False)
    ballot_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballots.id'), nullable=False)

    # Division between participations and votes maintains voter privacy
class ElectionParticipation(Base):
    __tablename__ = 'election_participation'
    ballot_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballots.id'), primary_key=True)
    voter_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), primary_key=True)


    ballot: Mapped["Ballot"] = relationship(back_populates='election_participations')
    user: Mapped["User"] = relationship(back_populates='election_participations')

class QuestionParticipation(Base):
    __tablename__ = 'question_participation'
    ballot_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballots.id'), primary_key=True)
    ballot_item_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballot_items.id'), primary_key=True)
    voter_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), primary_key=True)

    ballot: Mapped["Ballot"] = relationship(back_populates='question_participations')
    ballot_item: Mapped["BallotItem"] = relationship(back_populates='question_participations')
    user: Mapped["User"] = relationship(back_populates='question_participations')

class ReferendumVote(Base):
    __tablename__ = 'referendum_votes'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    ballot_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballots.id'), nullable=False)
    ballot_item_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballot_items.id'), nullable=False)
    option: Mapped[str] = mapped_column(String, nullable=False)
    
 
    ballot: Mapped["Ballot"] = relationship(back_populates='referendum_votes')
    item: Mapped["BallotItem"] = relationship(back_populates='referendum_votes')

class ElectionVote(Base):
    __tablename__ = 'election_votes'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    ballot_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballots.id'), nullable=False)
    ballot_item_id: Mapped[int] = mapped_column(Integer, ForeignKey('ballot_items.id'), nullable=True)
    # Determine if a vote is blank by checking if ballot_item_id IS NULL
```

```python
# Form tampering detection with immediate response
stmt = select(Ballot).where(Ballot.id == form.election_id.data)
election = db.session.execute(stmt).scalar()
if not election:
    return handle_suspicious_activity(
        current_user.id,
        f"User {current_user.id} tampered with election id {form.election_id.data}",
        ongoing_ballots_ids
    )
```

**Feature Demonstration:**

#### Video Demo:  <URL https://youtu.be/6h2qGB7IVCc>

## 📊 Automated Reporting

**The Challenge:**

  1. Processing results automatically at the end of the ballot using Celery.
  2. Performing comprehensive result analysis and enhancing the data visualization using Plotly within JavaScript.

**Technical Solution:**

```python
# Once ballots are stopped, the result processing begins immediately
@shared_task(soft_time_limit=60, time_limit=120)
def end_ongoing_ballots():
    stmt = select(Ballot).where(and_(
                                 Ballot.status == 'ongoing',
                                 Ballot.end <= datetime.now()
                                ))
    to_end_ballots = db.session.execute(stmt).scalars().all()
    if not to_end_ballots:
        return
    
    # Immediate lock to prevent new votes
    for ballot in to_end_ballots:
        ballot.status = 'ended'

    try:
        db.session.commit()
        ballot_ids_to_process = [str(ballot.id) for ballot in to_end_ballots]
        current_app.logger.info(f"Locked ballots {', '.join(ballot_ids_to_process)} - starting processing")
    except Exception as e:
        db.session.rollback()
        current_app.logger.error(f"Failed to lock ballots: {str(e)}")
        return

    successfully_processed_ballot_ids = []
    for ballot in to_end_ballots:

        try:
            stmt = select(BallotElectorate.electorate).where(BallotElectorate.ballot_id == ballot.id)
            electorate = db.session.execute(stmt).scalar()

            if ballot.type == "Election":

                stmt = select(func.count(ElectionParticipation.ballot_id)).where(ElectionParticipation.ballot_id == ballot.id)
                total_votes = db.session.execute(stmt).scalar()

                turnout = round(float(total_votes / electorate) * 100, 2)
                
                stmt = select(func.count(ElectionVote.id)).where(and_(
                    ElectionVote.ballot_id == ballot.id,
                    ElectionVote.ballot_item_id.is_not(None)
                ))
                valid_votes = db.session.execute(stmt).scalar()
                if total_votes > 0:
                    valid_votes_percentage = round(float(valid_votes / total_votes) * 100, 2)
                    invalid_votes_percentage = round(float((total_votes - valid_votes) / total_votes) * 100, 2)
                else:
                    valid_votes_percentage = 0.0
                    invalid_votes_percentage = 0.0
```

```javascript
// Demographic analysis with responsive pyramid chart using Plotly integration
function createPyramidChart(elementId, data) {
    const maxMale = Math.max(...data.male);
    const maxFemale = Math.max(...data.female);
    const maxValue = Math.max(maxMale, maxFemale);

    const reportSection = document.querySelector('.halved-report-section')
    const reportWidth = reportSection.offsetWidth

    const screenWidth = window.innerWidth;
    let chartWidth;
    if (screenWidth <= 900) {
        chartWidth = 750;
    } else {
        chartWidth = reportWidth - 80;
    }
    
    const maleTrace = {
        y: data.age_groups,
        x: data.male.map(val => -val),
        type: 'bar',
        orientation: 'h',
        name: 'Male',
        marker: { 
            color: '#3b82f6',
            line: { color: '#ffffff', width: 1 }
        },
        hovertemplate: '<b>Male</b><br>Age: %{y}<br>Count: %{customdata}<extra></extra>',
        customdata: data.male
    };
    
    const femaleTrace = {
        y: data.age_groups,
        x: data.female,
        type: 'bar',
        orientation: 'h',
        name: 'Female',
        marker: { 
            color: '#ec4899',
            line: { color: '#ffffff', width: 1 }
        },
        hovertemplate: '<b>Female</b><br>Age: %{y}<br>Count: %{x}<extra></extra>'
    };
    
    const layout = {
        barmode: 'overlay',
        xaxis: { 
            range: [-maxValue * 1.1, maxValue * 1.1],
            showgrid: false,
            zeroline: false,
            fixedrange: true,
            showline: false,
            tickmode: 'array',
            tickvals: maxValue <= 4 ? 
                [-maxValue, -Math.floor(maxValue/2), 0, Math.floor(maxValue/2), maxValue] :
                [
                    -maxValue, 
                    -maxValue * 0.75, 
                    -maxValue * 0.5, 
                    -maxValue * 0.25, 
                    0, 
                    maxValue * 0.25, 
                    maxValue * 0.5, 
                    maxValue * 0.75, 
                    maxValue
                ],
            ticktext: maxValue <= 4 ? 
                [maxValue.toString(), Math.floor(maxValue/2).toString(), '0', Math.floor(maxValue/2).toString(), maxValue.toString()] :
                [
                    Math.round(maxValue).toString(), 
                    Math.round(maxValue * 0.75).toString(), 
                    Math.round(maxValue * 0.5).toString(), 
                    Math.round(maxValue * 0.25).toString(), 
                    '0', 
                    Math.round(maxValue * 0.25).toString(), 
                    Math.round(maxValue * 0.5).toString(), 
                    Math.round(maxValue * 0.75).toString(), 
                    Math.round(maxValue).toString()
                ]
        },
        yaxis: { 
            categoryorder: 'array',
            categoryarray: data.age_groups.slice().reverse(),
            fixedrange: true,
            showgrid: false,
            showline: false
        },
        shapes: [
            ...data.age_groups.map((_, index) => ({
                type: 'line',
                x0: -maxValue * 1.1,
                x1: maxValue * 1.1,
                y0: index + 0.5,
                y1: index + 0.5,
                line: {
                    color: '#d1d5db',
                    width: 1
                }
            })).slice(0, -1),
            {
                type: 'line',
                x0: -maxValue * 1.1,
                x1: maxValue * 1.1,
                y0: -0.5,
                y1: -0.5,
                line: {
                    color: '#d1d5db',
                    width: 1
                }
            },
            {
                type: 'line',
                x0: -maxValue * 1.1,
                x1: maxValue * 1.1,
                y0: data.age_groups.length - 0.5,
                y1: data.age_groups.length - 0.5,
                line: {
                    color: '#d1d5db',
                    width: 1
                }
            },
            {
                type: 'line',
                x0: 0,
                x1: 0,
                y0: -0.7,
                y1: data.age_groups.length - 0.3,
                line: {
                    color: '#d1d5db',
                    width: 2
                },
                layer: 'above'
            }
        ],

        showlegend: false,
        legend: {
            orientation: 'h',
            x: 0.5,
            xanchor: 'center',
            y: -0.15
        },

        margin: { l: 35, r: 35, t: 50, b: 20 },
        plot_bgcolor: '#ffffff',
        paper_bgcolor: 'rgba(0,0,0,0)',
        width: chartWidth,
        height: 270,
        hoverlabel: {
        bgcolor: 'rgba(0, 0, 0, 0.8)',
        bordercolor: 'rgba(0, 0, 0, 0.8)',
        font: {
            color: 'white',
            family: 'Segoe UI, Tahoma, Geneva, Verdana, sans-serif',
            size: 12
        }
    },
    };
    
    const config = {
        displayModeBar: false,
        responsive: true,
        scrollZoom: false,
        doubleClick: false
    };
    
    Plotly.newPlot(elementId, [maleTrace, femaleTrace], layout, config);
}
```

**Feature Demonstration:**

#### Video Demo:  <URL https://youtu.be/qXZVnBM-2Zs>

## 🚀 Dynamic Initiative Platform

**Challenges:**

  1. Making civic engagement attractive.
  2. Encouraging creativity and providing a sense of accomplishment.
  3. The creation of a familiar, yet highly interactive environment encouraging debate and originality.
  4. Attaining a performance level inspired by modern social media.
  5. Creating multi-layer content moderation combining automation with administrative oversight.
  6. Establishing comprehensive filtering capabilites for the initiative feed page.

**Technical Solutions:**

The platform includes user profiles, connections(friendships) between users and personalized feeds, creating a complete social environment for civic participation.

![User Profile Interface](profile_readme.jpg)
*User profiles represent the initial environment for initiatives*

```python

# Model that allows social media level interactivity
class Initiative(Base):
    __tablename__ = 'initiatives'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    user_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), nullable=False)
    domain: Mapped[str] = mapped_column(String, nullable=False)
    document: Mapped[bytes] = mapped_column(LargeBinary, nullable=False)
    date: Mapped[datetime] = mapped_column(DateTime, default=datetime.now)
    title: Mapped[str] = mapped_column(String, nullable=False)
    subtitle: Mapped[str] = mapped_column(String, nullable=False)
    pinned: Mapped[bool] = mapped_column(Boolean, nullable=False, default=False)
    status: Mapped[str] = mapped_column(String, nullable=False, default="active")


    author: Mapped["User"] = relationship(back_populates='initiatives_created')
    likes: Mapped[List["Like"]] = relationship(back_populates='initiative')
    dislikes: Mapped[List["Dislike"]] = relationship(back_populates='initiative')
    hide_instances: Mapped[List["HiddenInitiative"]] = relationship(back_populates='initiative')
    promotions: Mapped[List["PromotedInitiative"]] = relationship(back_populates='initiative')
    comments: Mapped[List["Comment"]] = relationship(back_populates='initiative')

class Like(Base):
    __tablename__ = 'likes'
    initiative_id: Mapped[int] = mapped_column(Integer, ForeignKey('initiatives.id'), primary_key=True)
    user_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), primary_key=True)
    last_updated_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.now)


    __table_args__ = (
    PrimaryKeyConstraint('initiative_id', 'user_id'),
    )
    

    initiative: Mapped["Initiative"] = relationship(back_populates='likes')
    user: Mapped["User"] = relationship(back_populates='likes')
    
class Comment(Base):
    __tablename__ = 'comments'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    initiative_id: Mapped[int] = mapped_column(Integer, ForeignKey('initiatives.id'), primary_key=True)
    user_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), primary_key=True)
    text: Mapped[str] = mapped_column(Text, nullable=False)
    date: Mapped[datetime] = mapped_column(DateTime, default=datetime.now)
    status: Mapped[str] = mapped_column(String, nullable=False, default="active")
    

    initiative: Mapped["Initiative"] = relationship(back_populates='comments')
    author: Mapped["User"] = relationship(back_populates='comments')
    likes: Mapped["CommentLike"] = relationship(back_populates='comment')
    hide_instances: Mapped[List["HiddenComment"]] = relationship(back_populates='comment')
 # Additional models: Dislike, HiddenInitiative, PromotedInitiative, CommentLike, HiddenComment
 ```

 ``` javascript
// Infinite scrolling capabilities using bi-directional batch loading with memory management due to batch removal
function handleScroll() {
    if (isLoading) return;

    const scrollPosition = window.scrollY;
    const windowHeight = window.innerHeight;
    const documentHeight = document.documentElement.scrollHeight;

    const distanceFromBottom = documentHeight - (scrollPosition + windowHeight);
    const distanceFromTop = scrollPosition;

    if (distanceFromBottom < 2000 && lastInitiative < totalInitiatives) {
        const visibleCount = document.querySelectorAll('.initiative_card').length;

        if (visibleCount < 40) {
            loadBatch(lastInitiative - 20 + initialUnpinnedInitiatives, 20, 'after').then(() => {
                lastInitiative += 20;
                reinitializeForNewContent();
            });
        } else {
            loadBatch(lastInitiative - 20 + initialUnpinnedInitiatives, 20, 'after').then(() => {
                lastInitiative += 20;
                removeFirstBatch();
                reinitializeForNewContent();
            });
        }
    }

    if (distanceFromTop < 2000 && lastInitiative > 40) {
        loadBatch(lastInitiative - 80 + initialUnpinnedInitiatives, 20, 'before').then(() => {
            removeLastBatch();
            lastInitiative -= 20;
            reinitializeForNewContent();
        });
    }
}
 ```

``` python
# Automated content moderation techniques
@app.route("/create/comment", methods=["POST"])
@login_required
def create_comment():

    form = CreateCommentForm()
    if form.validate_on_submit():

        comment = form.comment.data

        is_valid, error_msg = validate_content(
        comment, 
        )

        comment_to_initiative_id = form.initiative_id.data

        if not is_valid:
            app.logger.warning(f"Inappropriate content detected in the comment of user {current_user.id} to the initiative {comment_to_initiative_id}.")
            return jsonify({"success": False, "message": f"{error_msg}"})

        stmt = select(Initiative).where(Initiative.id == comment_to_initiative_id)
        comment_to_initiative = db.session.execute(stmt).scalar()

        if not comment_to_initiative:
            app.logger.error(f"User {current_user.id} tried to submit a comment to the initiative {comment_to_initiative_id}, but it was not found.")
            return jsonify({"success": False, "message": "Initiative not found."})
        
        stmt = select(Comment).where(and_(
            Comment.user_id == current_user.id,
            Comment.initiative_id == comment_to_initiative_id,
            Comment.date > datetime.now() - timedelta(days=1)
        )).order_by(Comment.date.desc())
        comments_to_initiative_since_yesterday = db.session.execute(stmt).scalars().all()

        if len(comments_to_initiative_since_yesterday) >= 3:
            time_until_next_comment = comments_to_initiative_since_yesterday[-1].date - (datetime.now() - timedelta(days=1))
            app.logger.error(f"User {current_user.id} tried to submit more than 3 comments to the initiative {comment_to_initiative_id} in the last 24h and was refused.")
            return jsonify({"success": False, "message": format_cooldown_message(time_until_next_comment, 3, "24h", "comment")})  
        
        # form validation messages won't function for wtforms as this form doesn't render the template
        # we integrate them here
        comment = comment.strip()
        
        # HTML tags
        if '<' in comment or '>' in comment:
            return jsonify({"success": False, "message": "HTML tags not allowed"})
        
        # minimum meaningful content
        alpha_chars = sum(1 for c in comment if c.isalnum())
        if alpha_chars < 2:
            return jsonify({"success": False, "message": "Comment must contain at least 2 letters or numbers"})
        
        # repeating content
        if re.search(r'(.)\1{4,}', comment):
            return jsonify({"success": False, "message": "Excessive repeated characters not allowed"})
        
        # shouting
        if len(comment) > 10:
            caps_ratio = sum(1 for c in comment if c.isupper()) / len(comment)
            if caps_ratio > 0.7:
                return jsonify({"success": False, "message": "Excessive capitalization not allowed"})
        
        # special character abuse
        special_count = sum(1 for c in comment if not c.isalnum() and not c.isspace())
        if special_count > len(comment) * 0.3:
            return jsonify({"success": False, "message": "Too many special characters"})
        
        # troll text
        words = comment.split()
        for word in words:
            if len(word) > 40:
                return jsonify({"success": False, "message": "Words cannot exceed 40 characters"})

        new_comment = Comment(
            user_id = current_user.id,
            initiative_id = comment_to_initiative_id,
            text = comment
        )

        try:
            db.session.add(new_comment)
            db.session.commit()
            app.logger.info(f"User {current_user.id} submitted a comment to the initiative {comment_to_initiative_id}.")
            return jsonify({"success": True, "comment_id": new_comment.id}) # no message, visual effect will take place 
        except Exception as e:
            db.session.rollback()
            app.logger.error(f"User {current_user.id} encountered an error submiting a comment to the initiative {comment_to_initiative_id}: {str(e)}")
            return jsonify({"success": False, "message": "An error occurred during the process. Please try again."})                     

    return jsonify({"success": False, "message": "Invalid form data."})
 ```


 ```python
        # Overarching initiative filtering with personalization, domain validation, and social network integration
        hidden_initiatives_subquery = select(HiddenInitiative.initiative_id).where(
            HiddenInitiative.user_id == current_user.id
        )

        base_conditions = and_(
            Initiative.status == 'active',
            Initiative.id.not_in(hidden_initiatives_subquery),
            Initiative.user_id != current_user.id
        )

        if current_domains:
            for domain in current_domains:
                if domain not in [domain["key"] for domain in domain_data]:
                    return redirect(url_for('initiatives', sort=current_sort, search=search_term))
            base_conditions = and_(base_conditions, Initiative.domain.in_(current_domains))
                

        if search_term:
            search_filter = f"%{search_term}%"
            search_conditions = or_(
                Initiative.author.has(User.name.ilike(search_filter)),
                Initiative.author.has(User.surname.ilike(search_filter)),
                Initiative.title.ilike(search_filter)
            )
            base_conditions = and_(base_conditions, search_conditions)

        if current_sort == "newest_first":
            stmt = select(Initiative).options(
                joinedload(Initiative.author),
                joinedload(Initiative.comments).joinedload(Comment.author)
            ).where(base_conditions).order_by(Initiative.date.desc())
            
        elif current_sort == "most_popular":
            likes_subquery = select(
                Like.initiative_id,
                func.count().label('like_count')
            ).group_by(Like.initiative_id).subquery()
            
            stmt = select(Initiative).options(
                joinedload(Initiative.author),
                joinedload(Initiative.comments).joinedload(Comment.author)
            ).outerjoin(
                likes_subquery, Initiative.id == likes_subquery.c.initiative_id
            ).where(base_conditions).order_by(
                func.coalesce(likes_subquery.c.like_count, 0).desc(),
                Initiative.date.desc() 
            )

        elif current_sort == "most_commented":
            comments_subquery = select(
                Comment.initiative_id,
                func.count().label('comment_count')
            ).group_by(Comment.initiative_id).subquery()
            
            stmt = select(Initiative).options(
                joinedload(Initiative.author),
                joinedload(Initiative.comments).joinedload(Comment.author)
            ).outerjoin(
                comments_subquery, Initiative.id == comments_subquery.c.initiative_id
            ).where(base_conditions).order_by(
                func.coalesce(comments_subquery.c.comment_count, 0).desc(),
                Initiative.date.desc()
            )

        elif current_sort == "from_connections":
            friends_ids_query = union_all(
                select(Friendship.sender_id).where(and_(
                    Friendship.receiver_id == current_user.id, 
                    Friendship.status == True
                )),
                select(Friendship.receiver_id).where(and_(
                    Friendship.sender_id == current_user.id,
                    Friendship.status == True
                ))
            )
            friend_ids = db.session.execute(friends_ids_query).scalars().all()
            
            if friend_ids:
                friends_conditions = and_(base_conditions, Initiative.user_id.in_(friend_ids))
            else:
                friends_conditions = and_(base_conditions, Initiative.id == -1) # find nothing case because the query wouldn't work with an empty list
            
            stmt = select(Initiative).options(
                joinedload(Initiative.author),
                joinedload(Initiative.comments).joinedload(Comment.author)
            ).where(friends_conditions).order_by(Initiative.date.desc())
 ```

**Feature Demonstration**

#### Video Demo:  <URL https://youtu.be/i9K0UfOCrJU>

## 🔒 Multi-Layer Security

**Challenges:**

  1. Implementing admin level privilege controls with accountability, elevated threat protection, and reversible administrative actions to address potential abuse.
  2. Following OWASP best practices.
  3. Creating complete auditing capabilities using a logging system that registers every significant action and its author.
  4. Regulating user input by systematically employing WTForms.
  5. Implementing password re-verification before each sensitive action.
  6. Generating time limited tokens delivered by email to in the form of accessible links.

**Technical Solutions:**

```python
# Admin separation from normal users with security through obscurity to minimize attack surface exposure
def admin_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if current_user.rank == "admin":
            return f(*args, **kwargs)
        abort(404)
    
    return decorated_function
```

```python
# Admin account protection with straightforward unlocking procedure
# Employing rate limiting with OWASP recommended values, adapted to a platform of high responsibility
@app.route("/login", methods=["GET", "POST"])
@limiter.limit("100 per hour", methods=["GET"])
@limiter.limit("30 per hour", methods=["POST"])
@limiter.limit("100 per day, 10 per hour", key_func=limit_by_email, methods=["POST"])
# OWASP.org standard values
def login():
    form = LoginForm()
    
    if form.validate_on_submit():
        # Check if user is in pending approval queue
        stmt = select(Pending).where(Pending.email == form.email.data)
        user_in_pending = db.session.execute(stmt).scalar()
        
        if user_in_pending and check_password_hash(user_in_pending.hash, form.password.data):
            if user_in_pending.confirmed == False:
                app.logger.info(f"Login attempt for unconfirmed pending account {form.email.data}")
                flash("Check your email for account confirmation", "error")
            elif user_in_pending.confirmed == True:
                app.logger.info(f"Login attempt for confirmed but admin-pending account {form.email.data}")
                flash("User awaiting admin confirmation", "error")
            return render_template("login.html", form=form)
        
        # Check for existing user account
        stmt = select(User).where(User.email == form.email.data)
        user = db.session.execute(stmt).scalar()
        
        if not user:
            app.logger.warning(f"Failed login attempt for non-registered email: {form.email.data}")
            failed_login_attempt = FailedLoginAttempt(email=form.email.data)
            
            try:
                db.session.add(failed_login_attempt)
                db.session.commit()
                
                yesterday = datetime.now() - timedelta(days=1)
                stmt = select(func.count(FailedLoginAttempt.id)).where(and_(
                    FailedLoginAttempt.date >= yesterday,
                    FailedLoginAttempt.email == form.email.data
                ))
                failed_attempts = db.session.execute(stmt).scalar()
                
                if failed_attempts > 1:
                    delay_seconds = min(2 ** (failed_attempts - 1), 30)
                    time.sleep(delay_seconds)
                    
            except Exception as e:
                db.session.rollback()
                app.logger.error(f"Error: {str(e)} recording failed login attempt for non-registered email: {form.email.data}")
            
            flash("Wrong username or password", "error")
            return render_template("login.html", form=form)
        
        # Check for locked admin accounts
        if user.rank == "admin" and user.status == 'locked':
            app.logger.warning(f"Failed login attempt to admin {user.id} with locked account.")
            flash("Wrong username or password", "error")
            return render_template("login.html", form=form)
        
        # Handle incorrect password
        if not check_password_hash(user.hash, form.password.data):
            if user.rank == "admin":
                failed_login_attempt = FailedLoginAttempt(
                    email=user.email,
                    user_id=user.id
                )
                
                try:
                    db.session.add(failed_login_attempt)
                    db.session.commit()

                    yesterday = datetime.now() - timedelta(days=1)
                    stmt = select(func.count(FailedLoginAttempt.id)).where(and_(
                        FailedLoginAttempt.date >= yesterday,
                        FailedLoginAttempt.user_id == user.id
                    ))
                    failed_login_attempts = db.session.execute(stmt).scalar()
                    
                    app.logger.warning(f"Failed login attempt number {failed_login_attempts} to admin {user.id} within the last 24 hours.")
                    
                    if failed_login_attempts >= 5:
                        user.status = "locked"
                        db.session.commit()
                        app.logger.warning(f"Locked admin account {user.id} after {failed_login_attempts} login failures in the last 24 hours")
                        
                        # Generate unique unlock token
                        while(True):
                            token = secrets.token_urlsafe()
                            stmt = select(Token).where(Token.token == token)
                            duplicate_token = db.session.execute(stmt).scalar()
                            if not duplicate_token:
                                break
                        
                        deadline = datetime.now() + relativedelta(years=100)
                        unlock_link = f"https://civic-platform.org/unlock-account?token={token}"
                        
                        new_token = Token(
                            token=token,
                            email=user.email,
                            deadline=deadline,
                            type="unlock"
                        )
                        db.session.add(new_token)
                        
                        new_email = EmailInQueue(
                            address=user.email,
                            content=unlock_link,
                            type="unlock_account"
                        )
                        db.session.add(new_email)
                        db.session.commit()
                        
                        app.logger.info(f"Account unlock email queued for admin {user.id} with the token {token}")
                        
                except Exception as e:
                    db.session.rollback()
                    app.logger.error(f"Error: {str(e)} recording failed login attempt for admin {user.id}")
            
            else:
                app.logger.warning(f"Failed login attempt for user {user.id}")
                failed_login_attempt = FailedLoginAttempt(
                    email=user.email,
                    user_id=user.id
                )
                
                try:
                    db.session.add(failed_login_attempt)
                    db.session.commit()
                    
                    yesterday = datetime.now() - timedelta(days=1)
                    stmt = select(func.count(FailedLoginAttempt.id)).where(and_(
                        FailedLoginAttempt.date >= yesterday,
                        FailedLoginAttempt.email == form.email.data
                    ))
                    failed_attempts = db.session.execute(stmt).scalar()
                    
                    if failed_attempts > 1:
                        delay_seconds = min(2 ** (failed_attempts - 1), 30)
                        time.sleep(delay_seconds)
                        
                except Exception as e:
                    db.session.rollback()
                    app.logger.error(f"Error: {str(e)} recording failed login attempt for user {user.id}")
            
            flash("Wrong username or password", "error")
            return render_template("login.html", form=form)
        
        # Successful login
        try:
            login_user(user)
            session['tries'] = 0
            current_user.active = True
            
            if current_user.status == "deactivated":
                current_user.status = "in_use"
                app.logger.info(f"User {user.id} successfully reactivated their account")
            
            db.session.commit()
            app.logger.info(f"User {user.id} logged in successfully")
            
            next_page = request.args.get('next')
            return redirect(next_page or "/voting")
            
        except Exception as e:
            db.session.rollback()
            app.logger.error(f"Error during login process for {form.email.data}: {str(e)}")
            flash("An error occurred during login. Please try again.", "error")
    
    return render_template("login.html", form=form)
```

```python
# Admin deletions are effective, but reversible

# Initiative deletion by admins
initiative_to_delete.status = "deleted"

# Initiative selection includes only active ones
base_conditions = and_(
    Initiative.status == 'active',
    Initiative.id.not_in(hidden_initiatives_subquery),
    Initiative.user_id != current_user.id
)
```

```python
# Prompt action against form tampering with auditing, while maintaining database integrity
def handle_suspicious_activity(user_id, message, ballot_ids):

    try:
        stored_user_id = user_id
        for id in ballot_ids:
            stmt = select(Ballot).where(Ballot.id == id)
            ballot = db.session.execute(stmt).scalar()
            if ballot.type == 'Election':
                new_election_participation = ElectionParticipation(
                    voter_id = user_id,
                    ballot_id = id
                )
                new_election_vote = ElectionVote(
                    ballot_id = id,
                )
                db.session.add(new_election_vote)
                db.session.add(new_election_participation)
            else:
                for item in ballot.items:
                    new_question_participation = QuestionParticipation(
                        ballot_id = id,
                        ballot_item_id = item.id,
                        voter_id = user_id
                    )
                    new_question_vote = ReferendumVote(
                        ballot_id = id,
                        ballot_item_id = item.id,
                        option = 'BLANK'
                    )
                    db.session.add(new_question_vote)
                    db.session.add(new_question_participation)
        db.session.commit()
        logout_user()
        
        flash("Suspicious activity detected", "error")
        
        current_app.logger.warning(message)
        
        return redirect(url_for('login'))
    except Exception as e:
        current_app.logger.error(f"Failed to log out user {stored_user_id}: {str(e)}")
        return redirect(url_for('login'))
```

```python
# Making use of WTForms's inbuilt CSRF token, while improving UX via custom error messages for each field and establishing a reliable security standard for passwords
class RegisterForm(FlaskForm):
    sex = SelectField('Title', choices=[
        ('', 'Select title'),
        ('female', 'Mrs'),
        ('female', 'Ms'),
        ('male', 'Mr'),
        ('female', 'Miss')
    ], validators=[DataRequired(message="You have not selected your title")])
    
    name = StringField('First Name', validators=[
        DataRequired(message="You have not inserted your first name"),
        Length(max=50)
    ])
    
    surname = StringField('Last Name', validators=[
        DataRequired(message="You have not inserted your last name"),
        Length(max=50)
    ])
    
    email = EmailField('Email', validators=[
        DataRequired(message="You have not inserted your e-mail"),
        Email(message="Invalid email format"),
        Length(max=100)
    ])
    
    password = PasswordField('Password', validators=[
        DataRequired(message="You have not inserted a password"),
        Length(max=50)
    ])
    
    confirmation = PasswordField('Confirm Password', validators=[
        DataRequired(message="You have not inserted the password confirmation"),
        EqualTo('password', message="Password and confirmation do not match")
    ])
    
    birth = DateField('Birth Date', format='%Y-%m-%d', validators=[
        DataRequired(message="You have not chosen your birth date")
    ])
    
    document = FileField('ID Document', validators=[
        FileRequired(message="You have not uploaded a document"),
        FileAllowed(['pdf'], message="Document must be a PDF file"),
        FileSizeLimit(5)
    ])
    
    submit = SubmitField('Register')
    
    def validate_password(self, field):
        score = zxcvbn(field.data)["score"]
        if score < 3:
            suggestions = " ".join(zxcvbn(field.data)["feedback"]["suggestions"])
            raise ValidationError(f"Password too weak. {suggestions}")
        
    def validate_birth(self, field):
        birth_date = field.data
        min_birth_date = datetime.now().date() - relativedelta(years=18)
        if birth_date > min_birth_date:
            raise ValidationError(f"You must be at least 18 years old to register.")
```

```python
# Mandatory password re-verification for actions bearing consequences
@app.route("/reset/password", methods=["GET", "POST"])
@login_required
def reset_password():

    form = ResetPasswordLoggedInForm()

    if "tries" not in session:
        session["tries"] = 0

    if form.validate_on_submit():
        password = form.password.data
        if not check_password_hash(current_user.hash, password):
            session["tries"] += 1
            if session["tries"] >= 3:
                user_id = current_user.id
                try:
                    logout_user()
                    app.logger.warning(f"User {user_id} was disconnected after entering the wrong password 3 times in a row while logged in")
                    flash("Suspicious activity detected", "error")
                    return redirect(url_for('login'))
                except Exception as e:
                    app.logger.error(f"Failed to log out user {user_id}: {str(e)}")
            else:
                app.logger.warning(f"User {current_user.id} entered the wrong password for the {session['tries']}{'st' if session['tries'] == 1 else 'nd' if session['tries'] == 2 else 'rd' if session['tries'] == 3 else 'th'} time")
                flash(f"Wrong Password, {3 - session['tries']} {'tries' if (3 - session['tries']) > 1 else 'try'} remaining", "error")
                return render_template("reset_password.html", form=form)

        session["tries"] = 0

        hash = generate_password_hash(form.new_password.data)
        current_user.hash = hash
        db.session.commit()
        app.logger.info(f"User {current_user.id} successfully reset password")
        flash("Password successfully updated", "success")
        return redirect(url_for('login'))

    else:
        return render_template("reset_password.html", form=form)
```

```python
# Generating a 15-minute password reset token with collision protection, limited at maximum 3 for every 24h
# Prevents email enumeration attacks due to inconclusive flash message
    yesterday = datetime.now() - timedelta(days=1)
    stmt = select(Token).where(
            and_(
                Token.time_of_use >= yesterday,
                Token.email == email,
                Token.type == "recovery"
                )
            )
    recovery_tokens = db.session.execute(stmt).scalars().all()
    if len(recovery_tokens) >= 3:
        app.logger.warning(f"Password change rate limit reached for email: {email}")
        flash("Limit of 3 password change links per 24 hours reached.", "error")
        return render_template("email_for_recovery.html", form=form)
    while(True):
        token = secrets.token_urlsafe()
        stmt = select(Token).where(Token.token == token)
        duplicate_token = db.session.execute(stmt).scalar()
        if not duplicate_token:
            break
    deadline = datetime.now() + timedelta(minutes=15)
    reset_link = f"https://civic-platform.org/create/new-password?token={token}"
    try:
        new_email = EmailInQueue(
            address=email,
            content=reset_link,
            type="password_reset"
        )
        db.session.add(new_email)
        new_token = Token(
            token = token,
            email = email,
            deadline = deadline,
            type = "recovery"
        )
        db.session.add(new_token)
        db.session.commit()
        app.logger.info(f"Password reset email queued for user {is_email.id} with the token {token}")
        flash("If an account exists with that email, a password reset link has been sent.", "success")
        return redirect(url_for("login"))
```

## 💬 Real-Time Communication

**Challenges:**

  1. Enhancing user interaction by creating an encrypted, 1 to 1 real-time communication environment that feels intuitive and familiar.
  2. Developing competitive features that motivate users to opt for it instead of other counterparts.

**Technical Solutions:**

```python
# Connection-based messaging
class Friendship(Base):
    __tablename__ = 'friendships'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    sender_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), nullable=False)
    receiver_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), nullable=False)
    status: Mapped[bool] = mapped_column(Boolean, nullable=False, default=False)
    key: Mapped[Optional[str]] = mapped_column(String)
    # Encryption key generated upon friendship acceptance

   
    sender: Mapped["User"] = relationship(back_populates='friendships_sent', foreign_keys=[sender_id])
    receiver: Mapped["User"] = relationship(back_populates='friendships_received', foreign_keys=[receiver_id])

# Rich messaging with multimedia support and read receipts
class Message(Base):
    __tablename__ = 'messages'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    sender_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), nullable=False)
    receiver_id: Mapped[int] = mapped_column(Integer, ForeignKey('users.id'), nullable=False)
    text: Mapped[Optional[str]] = mapped_column(Text)
    date: Mapped[datetime] = mapped_column(DateTime, default=datetime.now)
    client_date: Mapped[str] = mapped_column(String, nullable=False)
    seen: Mapped[bool] = mapped_column(Boolean, nullable=False, default=False)
  
    sender: Mapped["User"] = relationship(back_populates='messages_sent', foreign_keys=[sender_id])
    receiver: Mapped["User"] = relationship(back_populates='messages_received', foreign_keys=[receiver_id])
    images: Mapped[List["Image"]] = relationship(back_populates='message')
    pdfs: Mapped[List["PDF"]] = relationship(back_populates='message')

class Image(Base):
    __tablename__ = 'chat_images'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    message_id: Mapped[int] = mapped_column(Integer, ForeignKey('messages.id'), primary_key=True)
    image: Mapped[bytes] = mapped_column(LargeBinary, nullable=False)
    
   
    message: Mapped["Message"] = relationship(back_populates='images')

class PDF(Base):
    __tablename__ = 'chat_pdfs'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    message_id: Mapped[int] = mapped_column(Integer, ForeignKey('messages.id'), primary_key=True)
    pdf: Mapped[bytes] = mapped_column(LargeBinary, nullable=False)
    name: Mapped[str] = mapped_column(String, nullable=False)
   
    message: Mapped["Message"] = relationship(back_populates='pdfs')
```

```python
# Web sockets implementation
@socketio.on('message')
def handle_message(data):
    
    sender_id = data["sender_id"]
    receiver_id = data["receiver_id"]
    message = data.get("message")
    files = data.get("files", [])
    room = data["room"]
    date_str = data["date"]
    date_obj = datetime.fromisoformat(date_str.replace('Z', '+00:00'))
    pdf_filenames = data.get('pdf_filenames', [])

    try:
        validate_csrf(data.get('csrf_token'))
    except ValidationError:
        socketio.emit('error', {'message': 'Security validation failed'}, room=request.sid)
        return

    stmt = select(Friendship).where(or_(and_(
        Friendship.sender_id == sender_id,
        Friendship.receiver_id == receiver_id,
        Friendship.status == True
    ),and_(
        Friendship.sender_id == receiver_id,
        Friendship.receiver_id == sender_id,
        Friendship.status == True
    )))
    is_friendship = db.session.execute(stmt).scalar()

    if is_friendship:

        key = is_friendship.key
        f = Fernet(key)
        
        encrypted_images = []
        encrypted_pdfs = []

        encrypted_message = f.encrypt(message.encode()) if message else None
        for file_data in files:
            if file_data.startswith('data:image/'):

                encrypted_images.append(f.encrypt(base64.b64decode(
                    add_padding(file_data.split(',')[1]))))
            elif file_data.startswith('data:application/pdf'):

                encrypted_pdfs.append(f.encrypt(base64.b64decode(
                    add_padding(file_data.split(',')[1]))))
        
        if not encrypted_message and not encrypted_images and not encrypted_pdfs:
            socketio.emit('error', {'message': 'You cannot submit an empty message.'}, room=request.sid)           
            return
        
        new_message=Message(
            sender_id=sender_id,
            receiver_id=receiver_id,
            text=encrypted_message,
            date=date_obj,
            client_date = date_str
        )
        try:
            db.session.add(new_message)
            db.session.commit()
            # no message, visual effect will take place
            # no app.logger for privacy
        except Exception as e:
            db.session.rollback()
            app.logger.error(f"Error: {str(e)} sending a message from {sender_id} to {receiver_id}")
            socketio.emit('error', {'message': 'An error occurred while sending the message. Please try again.'}, room=request.sid)
            return
            
        if encrypted_images:
            try:
                for encrypted_image in encrypted_images:
                    new_image = Image(
                        message_id=new_message.id,
                        image=encrypted_image
                    )
                    db.session.add(new_image)
                db.session.commit()
            except Exception as e:
                db.session.rollback()
                app.logger.error(f"Error: {str(e)} sending images from {sender_id} to {receiver_id}")
                socketio.emit('error', {'message': 'An error occurred while sending images. Please try again.'}, room=request.sid)
                return
            
        pdf_response_data = []
        if encrypted_pdfs:
            try:
                for i, encrypted_pdf in enumerate(encrypted_pdfs):
                    filename = pdf_filenames[i] if i < len(pdf_filenames) else 'document.pdf'
                    new_pdf = PDF(
                        message_id=new_message.id,
                        pdf=encrypted_pdf,
                        name=filename
                    )
                    db.session.add(new_pdf)
                    db.session.commit()
                    pdf_response_data.append({
                        'id': new_pdf.id,
                        'message_id': new_message.id
                    })
            except Exception as e:
                db.session.rollback()
                app.logger.error(f"Error: {str(e)} sending PDFs from {sender_id} to {receiver_id}")
                socketio.emit('error', {'message': 'An error occurred while sending PDFs. Please try again.'}, room=request.sid)
                return

        try:
            stmt = update(Message).where(
                and_(
                    Message.sender_id == receiver_id,
                    Message.receiver_id == sender_id,
                    Message.seen == False
                )
            ).values(seen=True)
            
            db.session.execute(stmt)
            db.session.commit()
            
            stmt = select(Message.id).where(
                and_(
                    Message.sender_id == receiver_id,
                    Message.receiver_id == sender_id,
                    Message.seen == True
                )
            )
            seen_message_ids = db.session.execute(stmt).scalars().all()
            
            if seen_message_ids:
                socketio.emit('seen_update', {
                    'message_ids': seen_message_ids,
                    'seen_by': sender_id
                }, room=room)
                
        except Exception as e:
            db.session.rollback()
            app.logger.error(f"Error: {str(e)} auto-updating seen status from {receiver_id} to {sender_id}")
            socketio.emit('error', {'message': 'An error occurred while updating message status. Please try again.'}, room=request.sid)
            return

        response_data = data.copy()
        response_data['pdf_ids'] = pdf_response_data
        response_data['message_id'] = new_message.id
        socketio.emit('message', response_data, room=room)

    else: 
        socketio.emit('error', {'message': 'You are not authorized to send messages in this chat.'}, room=request.sid)
            
@socketio.on('seen')
def handle_seen(data):
    
    try:
        validate_csrf(data.get('csrf_token'))
    except ValidationError:
        socketio.emit('error', {'message': 'Security validation failed'}, room=request.sid)
        return

    receiver_id = data['receiver_id']
    sender_id = data['sender_id']
    room = data['room']

    try:
        stmt = update(Message).where(
            and_(
                Message.sender_id == sender_id,
                Message.receiver_id == receiver_id,
                Message.seen == False
            )
        ).values(seen=True)
        
        result = db.session.execute(stmt)
        db.session.commit()
        
        if result.rowcount > 0:
            stmt = select(Message.id).where(
                and_(
                    Message.sender_id == sender_id,
                    Message.receiver_id == receiver_id,
                    Message.seen == True
                )
            )
            message_ids = db.session.execute(stmt).scalars().all()
            
            socketio.emit('seen_update', {
                'message_ids': message_ids,
                'seen_by': receiver_id
            }, room=room)
            
    except Exception as e:
        db.session.rollback()
        app.logger.error(f"Error: {str(e)} updating seen status from {sender_id} to {receiver_id}")
        socketio.emit('error', {'message': 'An error occurred while updating message status. Please try again.'}, room=request.sid)
        return


@socketio.on('join')
def on_join(data):
    room = data['room']
    join_room(room)
```

**Feature Demonstration**

#### Video Demo:  <URL https://youtu.be/RJBec5v-wxs>

## 📧 Automated Email Delivery Hub

**Challenges:**

  1. Providing crucial support in features like account creation, recovery, action confirmation and notifications.
  2. Creating an email queue that prevents bottlenecks, performs multiple delivery attempts and records errors.
  3. Designing custom email templates that contribute to the professional branding of CIVIC.

**Technical Solutions:**

```python
# Periodic celery task
@shared_task
def process_email_queue():
   stmt = select(EmailInQueue).where(
       or_(
           EmailInQueue.last_attempt == None,
           and_(
               EmailInQueue.attempts < 3,
               EmailInQueue.last_attempt < datetime.now() - timedelta(hours=1)
           )
       )
   )
   emails_to_process = db.session.execute(stmt).scalars().all()
   
   if not emails_to_process:
       return "No emails to process"
   
   for email in emails_to_process:
       email.attempts += 1
       email.last_attempt = datetime.now()
       success = False
       
       if email.type == "password_reset":
           try:
               send_password_reset_link(email.address, email.content)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending password reset link {email.content} to the user with the address {email.address}.")
       
       elif email.type == "account_validation":
           try:
               send_account_validation_link(email.address, email.content)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending account validation link {email.content} to the user with the address {email.address}.")
       
       elif email.type == "enrollment_accepted":
           try:
               enrollment_accepted(email.address)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending enrollment accepted email to the user with the address {email.address}.")
       
       elif email.type == "enrollment_rejected":
           try:
               enrollment_rejected(email.address, email.content)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending enrollment rejected email with the reason {email.content} to the user with the address {email.address}.")
       
       elif email.type == "unlock_account":
           try:
               send_unlock_account_link(email.address, email.content)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending account unlock link {email.content} to the user with the address {email.address}.")
       
       elif email.type == "deleted_initiative_notification":
           try:
               initiative_deleted_by_admin_notification_email(email.address, email.content, email.content2)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending initiative deletion notification with the reason {email.content2} to the user with the address {email.address}.")
       
       elif email.type == "connection_request_received_notification":
           try:
               connection_request_received_notification_email(email.address, email.content, email.content2, email.content3)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending connection request received notification from the user {email.content} to the user with the address {email.address}.")
       
       elif email.type == "connection_request_accepted_notification":
           try:
               connection_request_accepted_notification_email(email.address, email.content, email.content3)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending connection request accepted notification to the user with the address {email.address}.")
       
       elif email.type == "initiative_promotion_notification":
           try:
               initiative_promotion_notification_email(email.address, email.content, email.content2, email.content3, email.content4)
               success = True
           except Exception as e:
               current_app.logger.error(f"Error: {str(e)} sending initiative promotion notification by the user {email.content} to the user with the address {email.address}.")
       
       elif email.type == "deleted_comment_notification":
           try:
               comment_deleted_by_admin_notification_email(email.address, email.content, email.content2)
               success = True
           except Exception as e:
               current_app.logger.error(f'Error: {str(e)} sending comment deletion notification for the initiative with the title "{email.content2}" to the user with the address {email.address}.')
       
       if success == True:
           db.session.delete(email)
       
       if email.attempts >= 3:
           current_app.logger.warning(f"Email {email.id} to {email.address} failed 3 times, will stop retrying")
   
   db.session.commit()
```

```python
# Email template example
def enrollment_accepted(email):
    subject = "Enrollment accepted"
    
    html_body = """<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to CIVIC</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            color: #333333;
            margin: 0;
            padding: 0;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        .header {
            background-color: #2455c6;
            padding: 20px;
            text-align: center;
        }
        .header h1 {
            color: white;
            margin: 0;
            font-size: 24px;
            margin-right:15px
        }
        .content {
            padding: 20px;
            background-color: #ffffff;
        }
        .button {
            display: inline-block;
            background-color: #2455c6;
            color: white;
            text-decoration: none;
            padding: 12px 24px;
            border-radius: 4px;
            margin: 20px 0;
            font-weight: bold;
        }
        .footer {
            text-align: center;
            padding: 20px;
            font-size: 12px;
            color: #666666;
            background-color: #f7f7f7;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Welcome to CIVIC</h1>
      
            </div>
        <div class="content">
            <p>Good news! Your request for a CIVIC account has been accepted by an admin.</p>
            
            <p>You are now officially part of our community. We're excited to have you join us!</p>
            
            <div style="text-align: center;">
                <a href="https://civic-platform.org/login" class="button" style="color:white">Log In Now</a>
            </div>
            
            <p>If you have any questions about getting started, please don't hesitate to reach out to our support team.</p>
            
            <p>We hope to see you soon!</p>
            
            <p>Best regards,<br>The CIVIC Support Team</p>
        </div>
        <div class="footer">
            <p>© 2025 CIVIC. All rights reserved.</p>
            <p>This email was sent to you because you registered for a CIVIC account.</p>
        </div>
    </div>
</body>
</html>"""
    
    text_body = """Your request for a CIVIC account has been accepted by an admin. Welcome to the community!

Log In: https://civic-platform.org/login

We hope to see you soon,
The CIVIC Support team"""
    
    msg = MIMEMultipart("mixed")
    msg["Subject"] = subject
    msg["From"] = smtp_username
    msg["To"] = email
    
    alt = MIMEMultipart("alternative")

    part1 = MIMEText(text_body, "plain")
    alt.attach(part1)
    
    part2 = MIMEText(html_body, "html")
    alt.attach(part2)

    msg.attach(alt)
    
    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.starttls()
        server.login(smtp_username, smtp_password)
        server.sendmail(smtp_username, email, msg.as_string())
```

### CIVIC demonstrates production-ready full-stack development with enterprise-level architecture:

**Technical Stack**: Python/Flask, JavaScript, SQLAlchemy, Celery, WebSockets, SMTP

**Live Demo**: [https://civic-platform.org](https://civic-platform.org)

